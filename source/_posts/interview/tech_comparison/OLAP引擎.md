



[toc]

# doris和clickhouse全方位对比

### 一、核心定位与架构

- **Apache Doris**：Apache 顶级项目，**标准 MPP 架构**（FE+BE），无本地 / 分布式表区分，统一逻辑表，兼容 MySQL 协议与 SQL。
- **ClickHouse**：ClickHouse Inc. 主导，**Shared-Nothing+Scatter-Gather**，分 Local 表与 Distributed 表，自研 SQL 方言，依赖 ZooKeeper。

### 二、数据模型与更新能力

|    维度     |                         Apache Doris                         |                          ClickHouse                          |
| :---------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  数据模型   | **3 种模型**：Duplicate (明细)、Aggregate (聚合)、Unique (主键更新) | **MergeTree 家族**：ReplacingMergeTree、SummingMergeTree 等，模型单一 |
| 事务 / 更新 | **强一致事务**，支持**同步 UPSERT/DELETE**，主键更新性能比 CK 快**18–34 倍**VeloDB | **无完整事务**，更新异步（后台 Merge），存在读写不一致，主键更新极慢 |
| Schema 变更 |                动态调整，支持在线 DDL，成本低                |                    变更成本高，常需重建表                    |

### 三、查询引擎与性能（2026 基准）

- **单表查询**：ClickHouse 略优（极致压缩 + 向量化）；Doris 接近，ClickBench 互有胜负。

- 多表 Join ：Doris 碾压级优势

  - Doris：成熟**CBO 优化器**，支持 Shuffle/Broadcast/Colocate Join、Runtime Filter；TPC-H 比 CK 快**60 倍**Apache Doris。
  - ClickHouse：无 CBO，Join 能力弱，复杂查询需 SQL 改写或打平宽表Apache Doris。

  

- **并发能力**：Doris 支持**千级 QPS**（BI / 报表）；ClickHouse 并发低，高负载易引发 Merge 风暴Apache Doris。

- **实时更新查询**：Doris（Unique Key）比 CK（ReplacingMergeTree）快**2.5–4.6 倍**（ClickBench）VeloDB。

![img](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27256%27%20height=%27192%27/%3e)![image](https://p26-flow-imagex-sign.byteimg.com/labis/image/5483818733d6718e0dc3551c936762b7~tplv-a9rns2rl98-pc_smart_face_crop-v1:470:352.image?lk3s=8e244e95&rcl=202605122230597D5AF6C5482CA777B2FD&rrcfp=cee388b0&x-expires=2093956274&x-signature=oE%2F2wZfUSdwA1FHsfBbZpNScGL8%3D)

![img](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27256%27%20height=%27192%27/%3e)![image](https://p6-flow-imagex-sign.byteimg.com/labis/image/ae4fea46a42248f6e17d0dc7ee2f98e9~tplv-a9rns2rl98-pc_smart_face_crop-v1:470:352.image?lk3s=8e244e95&rcl=202605122230597D5AF6C5482CA777B2FD&rrcfp=cee388b0&x-expires=2093956274&x-signature=bPCfQ63WzJcozkaJIZmSNxnIsYA%3D)

![img](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%27256%27%20height=%27192%27/%3e)![image](https://p11-flow-imagex-sign.byteimg.com/labis/image/822f853f50f7e42619927864e8637fc8~tplv-a9rns2rl98-pc_smart_face_crop-v1:470:352.image?lk3s=8e244e95&rcl=202605122230597D5AF6C5482CA777B2FD&rrcfp=cee388b0&x-expires=2093956274&x-signature=ndyCPGLtG9BVn0UewoAwZ2DAJE8%3D)

### 四、写入与实时性

- Doris：

  - 写入：Stream Load/Flink CDC/Routine Load，支持**事务性导入**（Exactly-Once）。
  - 实时：亚秒级摄入，支持 MySQL Binlog 实时同步，高并发写入稳定。

  

- ClickHouse：

  - 写入：批量（INSERT/CSV）性能强；并发写易触发 Merge，**性能陡降**。
  - 实时：依赖 Kafka+MaterializedView，延迟高，更新冲突多。

  

### 五、SQL 兼容性与易用性

- **Doris**：**高度兼容 MySQL**（协议 + 语法），支持子查询 / CTE / 窗口函数，学习成本低。
- **ClickHouse**：**非标 SQL 方言**，窗口函数 / 关联语法特殊，需适配；Local+Distributed 表分离，需手动路由。

### 六、运维与扩缩容

- Doris：

  - 无外部依赖（无 ZooKeeper），FE/BE 自管理。
  - **自动扩缩容 + 副本平衡**，故障自动转移，运维极简Apache Doris。

  

- ClickHouse：

  - 强依赖 ZooKeeper，部署 / 配置复杂（XML）。
  - 扩容需手动平衡数据，新节点不自动接管负载，运维成本高。

  

### 七、湖仓一体与生态

- **Doris**：原生支持**Hive/Iceberg/Hudi/Paimon**，可直接查询数据湖，物化视图加速湖仓查询。
- **ClickHouse**：湖仓集成弱，需自定义开发，兼容性差。

### 八、适用场景

#### ✅ 选 Apache Doris

- **混合负载**：实时报表 + BI 看板 + 即席查询（高并发）。
- **复杂多表关联**：星型 / 雪花模型分析、用户行为路径分析。
- **实时更新**：交易数据、用户画像、MySQL Binlog 同步。
- **湖仓一体**：数据湖查询、流批一体分析。
- **低运维**：团队小，希望 “MySQL 式” 易用性。

#### ✅ 选 ClickHouse

- **单表海量分析**：PB 级日志、广告点击流、时序数据（大宽表）。
- **极致压缩存储**：成本敏感，数据极少更新。
- **离线批量**：ETL 后大宽表聚合，无复杂 Join。
- **强定制团队**：有能力维护 ZooKeeper、调优 Merge、处理读写不一致。

### 九、选型总结（一句话）

- **Doris**：**全能型 OLAP**，强在**多表 Join、实时更新、高并发、易用运维、湖仓一体**，适合企业级混合负载与实时分析。
- **ClickHouse**：**单表性能之王**，强在**极致单表查询、高压缩**，适合日志 / 时序 / 离线宽表场景，需接受高运维成本与弱 Join 能力。