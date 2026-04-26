

[toc]



两个Openwrt路由器怎么有线桥接









## 当副路由器频繁出现无ip分配，怎么解决

我发现当进入副路由器设置，接口 -> lan -> 点击连接，可以重新分配ip

### 有效方案

思路：通过日志判断设备连接是否异常，如果异常就重新连接lan口

脚本：

```shell
#!/bin/sh

# ==================== 用户配置区域 ====================
# 监控的关键词（正则表达式，可根据需要增删）
# 例如：无线客户端因不活动被踢、AP 断开、DHCP 丢失等
KEYWORDS="deauthenticated due to inactivity|AP-STA-DISCONNECTED|no lease|DHCP lease lost|link down"

# 触发后的冷却时间（秒），防止频繁恢复
COOLDOWN=600

# 恢复动作命令（根据需要选择一种，并注释掉其他）
# 选项1：重启指定网络接口（例如 lan 或 wan）
# ACTION="ubus call network.interface.lan down && sleep 2 && ubus call network.interface.lan up"
# 选项2：重启整个无线服务（影响所有 WiFi 客户端）
# ACTION="wifi"
# 选项3：重启整个网络（较重量级）
# ACTION="/etc/init.d/network restart"
# 选项4：使用openWrt封装的接口操作命令，重新连接接口
ACTION="ifup lan"

# 持久化统计文件（记录触发次数和最近时间）
STATS_FILE="/root/monitor_stats.txt"

# 详细日志文件（记录每次触发的信息）
LOG_FILE="/tmp/monitor.log"

# ==================== 初始化统计 ====================
if [ ! -f "$STATS_FILE" ]; then
    echo "0" > "$STATS_FILE"          # 初始触发次数为0
    echo "Never" >> "$STATS_FILE"      # 上次触发时间
fi

# ==================== 实时监控循环 ====================
logread -f | while read line; do
    # 将每一行日志写入详细日志文件（可选，可注释掉以节省空间）
    echo "$(date '+%Y-%m-%d %H:%M:%S'): $line" >> "$LOG_FILE"

    # 检查是否匹配关键词
    if echo "$line" | grep -qE "$KEYWORDS"; then
        current=$(date +%s)

        # 读取上次触发时间（从文件获取）
        last_trigger_time=$(sed -n '2p' "$STATS_FILE" 2>/dev/null)
        if [ "$last_trigger_time" = "Never" ] || [ -z "$last_trigger_time" ]; then
            last_ts=0
        else
            # 将上次触发时间转换为时间戳（假设格式为 YYYY-MM-DD HH:MM:SS）
            last_ts=$(date -d "$last_trigger_time" +%s 2>/dev/null || echo 0)
        fi

        # 检查冷却时间
        if [ $((current - last_ts)) -gt $COOLDOWN ] || [ $last_ts -eq 0 ]; then
            # 读取当前触发次数，增加并写回
            count=$(head -1 "$STATS_FILE")
            new_count=$((count + 1))
            current_time=$(date '+%Y-%m-%d %H:%M:%S')
            echo "$new_count" > "$STATS_FILE"
            echo "$current_time" >> "$STATS_FILE"

            # 记录详细日志
            echo "$current_time - Triggered by: $line" >> "$LOG_FILE"
            logger -t "network_monitor" "Triggered action due to: $line (total: $new_count)"

            # 执行恢复动作
            eval "$ACTION"

            # 可选：等待几秒确保动作完成
            sleep 2
        else
            # 冷却期内不触发，但记录日志
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Suppressed (cooldown): $line" >> "$LOG_FILE"
        fi
    fi
done
```

优化脚本，增加一些输出日志

```shell
#!/bin/sh

# ==================== 配置区域 ====================
KEYWORDS="deauthenticated due to inactivity|AP-STA-DISCONNECTED|no lease|DHCP lease lost|link down"
COOLDOWN=600                    # 冷却时间（秒）
ACTION="ifup lan"                    # 恢复动作（可根据需要修改）
COOLDOWN_FILE="/tmp/monitor_last.txt"   # 用于冷却判断：第一行计数，第二行上次触发时间
HISTORY_FILE="/root/monitor_history.txt" # 存储所有触发时间，每行一个
STATS_FILE="/root/monitor_stats.txt"     # 最终输出的统计文件（含间隔）
LOG_FILE="/tmp/monitor.log"             # 详细日志

# ==================== 初始化 ====================
# 初始化冷却文件
if [ ! -f "$COOLDOWN_FILE" ]; then
    echo "0" > "$COOLDOWN_FILE"
    echo "Never" >> "$COOLDOWN_FILE"
fi
# 初始化历史文件（空）
if [ ! -f "$HISTORY_FILE" ]; then
    touch "$HISTORY_FILE"
fi

# ==================== 函数：生成统计文件 ====================
generate_stats() {
    tmp_stats=$(mktemp)
    total=$(wc -l < "$HISTORY_FILE")
    echo "$total" > "$tmp_stats"

    prev_ts=0
    first=1
    while read -r time_str; do
        [ -z "$time_str" ] && continue
        ts=$(date -d "$time_str" +%s 2>/dev/null || echo 0)
        if [ "$first" -eq 1 ]; then
            diff=0
            first=0
        else
            diff=$(( (ts - prev_ts) / 60 ))   # 整数分钟
        fi
        echo "$time_str $diff" >> "$tmp_stats"
        prev_ts=$ts
    done < "$HISTORY_FILE"

    mv "$tmp_stats" "$STATS_FILE"
}

# ==================== 实时监控循环 ====================
logread -f | while read line; do
    echo "$(date '+%Y-%m-%d %H:%M:%S'): $line" >> "$LOG_FILE"
    if echo "$line" | grep -qE "$KEYWORDS"; then
        current=$(date +%s)
        current_time_str=$(date '+%Y-%m-%d %H:%M:%S')

        # 读取上次触发时间
        last_trigger_time=$(sed -n '2p' "$COOLDOWN_FILE" 2>/dev/null)
        if [ "$last_trigger_time" = "Never" ] || [ -z "$last_trigger_time" ]; then
            last_ts=0
        else
            last_ts=$(date -d "$last_trigger_time" +%s 2>/dev/null || echo 0)
        fi

        # 检查冷却时间
        if [ $((current - last_ts)) -gt $COOLDOWN ] || [ $last_ts -eq 0 ]; then
            # 更新冷却文件中的计数和时间
            count=$(head -1 "$COOLDOWN_FILE")
            new_count=$((count + 1))
            echo "$new_count" > "$COOLDOWN_FILE"
            echo "$current_time_str" >> "$COOLDOWN_FILE"

            # 记录到历史文件
            echo "$current_time_str" >> "$HISTORY_FILE"

            # 重新生成统计文件
            generate_stats

            # 记录详细日志并执行恢复动作
            echo "$current_time_str - Triggered by: $line" >> "$LOG_FILE"
            logger -t "network_monitor" "Triggered action due to: $line (total: $new_count)"
            eval "$ACTION"
            sleep 2
        else
            echo "$(date '+%Y-%m-%d %H:%M:%S') - Suppressed (cooldown): $line" >> "$LOG_FILE"
        fi
    fi
done
```



命令：

```shell
vi /root/network_monitor.sh   # 粘贴脚本内容
chmod +x /root/network_monitor.sh

# 测试运行效果 ctrl+c停止
/root/network_monitor.sh

# 设置开机自启动
# 编辑 /etc/rc.local，在 exit 0 之前添加：
/root/network_monitor.sh &


cat /root/monitor_stats.txt
tail -f /tmp/monitor.log
```







### 无效方案（仅记录）：

方案一：定时查询网络是否断开，如果是则重启

```shell
vim /root/check_network.sh
	``
  #!/bin/sh
  LOG_FILE="/tmp/network_check.log"
  TARGET="192.168.1.1"       # 主路由器 IP
  INTERFACE="br-lan"         # 要检查的接口，一般是 br-lan

  echo "$(date): Starting check" >> $LOG_FILE

  #1. 检查接口是否有 IP
  IP_ADDR=$(ip addr show $INTERFACE | grep -o 'inet [0-9.]*' | cut -d' ' -f2)
  if [ -z "$IP_ADDR" ]; then
      echo "$(date): No IP on $INTERFACE" >> $LOG_FILE
      NEED_RESTART=1
  else
      echo "$(date): Current IP $IP_ADDR" >> $LOG_FILE
  fi

  #2. 检查能否 ping 通主路由器
  if ! ping -c 2 -W 2 $TARGET > /dev/null 2>&1; then
      echo "$(date): Cannot ping $TARGET" >> $LOG_FILE
      NEED_RESTART=1
  else
      echo "$(date): Ping $TARGET OK" >> $LOG_FILE
  fi

  #3. 尝试 DHCP 续租（如果接口是通过 DHCP 获取 IP）
  #注意：如果接口是静态 IP，这一步可以跳过
  #这里假设 WAN 口负责 DHCP，如果没有 WAN 口则调整
  if [ "$NEED_RESTART" = "1" ]; then
      echo "$(date): Condition triggered, restarting network" >> $LOG_FILE
      /etc/init.d/network restart
      #或者只重启 WAN 口：ifdown wan && sleep 2 && ifup wan
  else
      echo "$(date): All checks passed" >> $LOG_FILE
  fi
  ``
  
chmod +x /root/check_network.sh
crontab -e
  ``
	*/5 * * * * /root/check_network.sh
	``

#检查crontab日志
logread -f | grep -i cron
```



后续发现，长时间连接的设备会出现这个问题，新加入的可以正常获取ip。根据无法连接的设备MAC，搜索了相关日志

```shell
logread | grep -i "A2:2C:D3:55:3F:2F"

Sat Feb 15 14:11:14 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f IEEE 802.11: authenticated
Sat Feb 15 14:11:14 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f IEEE 802.11: authenticated
Sat Feb 15 14:11:14 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f IEEE 802.11: associated (aid 2)
Sat Feb 15 14:11:14 2020 daemon.notice hostapd: wlan0: AP-STA-CONNECTED a2:2c:d3:55:3f:2f
Sat Feb 15 14:11:14 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f RADIUS: starting accounting session A2C964F56F596AAD
Sat Feb 15 14:11:14 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f WPA: pairwise key handshake completed (RSN)
Sat Feb 15 14:21:54 2020 daemon.notice hostapd: wlan0: AP-STA-DISCONNECTED a2:2c:d3:55:3f:2f
Sat Feb 15 14:21:54 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f IEEE 802.11: disassociated
Sat Feb 15 14:21:55 2020 daemon.info hostapd: wlan0: STA a2:2c:d3:55:3f:2f IEEE 802.11: deauthenticated due to inactivity (timer DEAUTH/REMOVE)
```

分析日志得出：

你提供的日志清晰地揭示了问题所在：**副路由器的无线接入点（hostapd）因为设备长时间不活动（inactivity），主动将设备踢下线了**。这正是“长时间连接的设备无法获取IP，而新设备可以”的根本原因。

- 日志显示，设备在 14:11:14 成功连接（authenticated, associated, key handshake completed）。
- 10分钟后（14:21:54），hostapd 记录 `AP-STA-DISCONNECTED` 和 `deauthenticated due to inactivity`，即因不活动超时被断开。
- 此时设备虽然认为自己还在 WiFi 上，但实际无线链路已被 AP 关闭。当设备需要 DHCP 续租时，发出的请求无法到达副路由器，自然也无法到达主路由器，因此获取不到 IP。

解决方案：

#### 方法一：通过 OpenWrt Web 界面（LuCI）修改（如果可访问）

1. 进入 **网络 → 无线**。
2. 找到你的无线网络（通常是 SSID 对应的那个 `AP` 模式接口），点击 **编辑**。
3. 切换到 **高级设置** 选项卡。
4. 找到 **“AP 客户端不活动超时”**（或 `AP Client Inactivity Timeout`），默认可能是 `300` 秒或 `0`（表示使用 hostapd 默认值）。将其改为较大的值，例如 `86400` 秒（24小时）或 `0` 表示禁用超时（不推荐完全禁用，但可设为很大的值）。
5. 点击 **保存&应用**。

#### 方法二：通过 SSH 命令行修改（推荐，更精确）

1. 登录副路由器的 SSH。

2. 编辑无线配置文件：

   bash

   复制下载

   ```
   vi /etc/config/wireless
   ```

   

3. 找到你的 `wifi-iface` 配置块（`option mode 'ap'` 的那个）。在其中添加（或修改）以下两行：

   bash

   复制下载

   ```
   option max_inactivity '86400'      # 不活动超时设为 24 小时（单位：秒）
   option disassoc_low_ack '0'        # 禁止因 ACK 丢失而断开（可选，可提高稳定性）
   ```

   

   示例配置段：

   bash

   复制下载

   ```
   config wifi-iface 'default_radio0'
           option device 'radio0'
           option mode 'ap'
           option ssid '你的WiFi名'
           option encryption 'psk2'
           option key '你的密码'
           option network 'lan'
           option max_inactivity '86400'
           option disassoc_low_ack '0'
   ```

   

4. 保存文件并退出（`:wq`）。

5. 重启无线服务使配置生效：

   bash

   复制下载

   ```
   wifi
   ```

   

   或者重启整个网络：

   bash

   复制下载

   ```
   /etc/init.d/network restart
   ```

   

------

#### 📝 补充说明

- **`max_inactivity` 作用**：指定客户端在不发送任何数据帧的情况下，AP 等待多少秒后将其断开。默认值通常为 300 秒（5分钟）。设为较大的值（如 86400）可确保设备即使长时间无流量也不会被踢。
- **`disassoc_low_ack`**：如果设为 `1`（默认），AP 会在多次未收到客户端 ACK 确认时将其断开。设为 `0` 可以避免因短暂信号波动导致的误踢，但可能会使“死客户端”停留更久。可根据环境决定是否启用。
- 调整后，之前被踢的设备重新连接后，将不会再因不活动而断开，从而能正常保持 IP 租约。