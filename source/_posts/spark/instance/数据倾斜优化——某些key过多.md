---
    title: 数据倾斜优化——某些key过多
    categories: spark
    tags:
    creator: cjq
    create_time: 2021/03/13



---

<details>
<summary>explode 与 lateral view 对比</summary>
<pre><code>
select user_coupon_id, explode(split('0,1', ',')) as tag
from mart_waimai.aggr_act_ord_use_coupon_dd
where dt='20200920'
limit 10
</code></pre>
</details>

## 背景

统计端维度的日活、下单UV、成交UV等，由于端信息只能从流量表里获取，所以需要从流量表关联订单表得到成交信息。

## 开发过程

### 原始sql

```sql
SELECT b.region_id,
       b.region_name,
       b.management_city_id,
       b.management_city_name,
       a.device_type_id,
       count(DISTINCT a.customer_id) AS dau,
       count(DISTINCT IF (a.event_type = 'view'
                          AND a.val_cid = 'page_csu_list', a.customer_id, NULL)) AS home_expose_uv,
       count(DISTINCT IF (a.event_type = 'click'
                          AND a.val_cid = 'page_csu_list', a.customer_id, NULL)) AS home_click_uv,
       count(DISTINCT IF (a.val_bid = 'b_h8ds56xuac', a.customer_id, NULL)) AS add_cart_uv,
       count(DISTINCT IF (a.val_cid = 'page_order_confirm'
                          AND a.val_bid = 'b_drqbsnud', a.customer_id, NULL)) AS order_uv,
       count(DISTINCT IF (c.order_id IS NOT NULL, a.customer_id, NULL)) AS order_arrange_uv,
       sum(IF (c.order_id IS NOT NULL, a.arranged_amt, 0)) AS conversion_sales_amt
FROM mart_caterb2b.fact_flow_visit_click_kv_day a
JOIN
  (SELECT m.dt,
          m.customer_id,
          n.region_id,
          n.region_name,
          n.management_city_id,
          n.management_city_name
   FROM mart_caterb2b.dim_caterb2b_customer_his m
   JOIN mart_caterb2b.dim_sm_management_city_info n ON m.city_id = n.city_id
   WHERE m.dt = '20210307') b ON a.customer_id = b.customer_id
AND a.dt = b.dt
LEFT JOIN
  (SELECT DISTINCT dt,
                   order_id,
                   parent_order_id
   FROM mart_caterb2b.fact_biz_order_day
   WHERE dt = '20210307') d ON a.order_id = d.parent_order_id
AND a.dt = d.dt
LEFT JOIN
  (SELECT dt,
          order_id,
          sum(arranged_amt) AS arranged_amt
   FROM mart_caterb2b.mid_deal_order_item_withpop
   WHERE dt = '20210307'
     AND is_arranged = 1
     AND channel_id = 1001
     AND order_type NOT IN (3,
                            4)-- 排除补货换货

     AND spu_type != 5 -- 排除包装物

   GROUP BY dt,
            order_id) c ON d.dt = c.dt
AND d.order_id = c.order_id
WHERE a.dt = '20210307'
GROUP BY b.region_id,
         b.region_name,
         b.management_city_id,
         b.management_city_name,
         a.device_type_id
```

出现数据倾斜

![](https://tva1.sinaimg.cn/large/008eGmZEly1goihhlc415j32q70go7af.jpg)

查看出问题stage

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goihhltxigj30u00ujwk5.jpg" style="zoom:50%;" />



定位sql

![](https://tva1.sinaimg.cn/large/008eGmZEly1goihhln8kvj30u00xjqgy.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1goihni8b87j30hl0edabr.jpg)

怀疑是因为d表dt很多关联不上为null，导致某些某些节点量特别大，所以考虑提前关联c表数据，如下：

```sql
SELECT b.region_id,
           b.region_name,
           b.management_city_id,
           b.management_city_name,
           a.device_type_id,
           count(DISTINCT a.customer_id) AS dau,
           count(DISTINCT IF (a.event_type = 'view'
                              AND a.val_cid = 'page_csu_list', a.customer_id, NULL)) AS home_expose_uv,
           count(DISTINCT IF (a.event_type = 'click'
                              AND a.val_cid = 'page_csu_list', a.customer_id, NULL)) AS home_click_uv,
           count(DISTINCT IF (a.val_bid = 'b_h856xuac', a.customer_id, NULL)) AS add_cart_uv,
           count(DISTINCT IF (a.val_cid = 'page_order_confirm'
                              AND a.val_bid = 'b_drqbsnud', a.customer_id, NULL)) AS order_uv,
           count(DISTINCT IF (a.val_cid = 'page_order_confirm'
                              AND a.val_bid = 'b_drqbsnud'
                              AND d.order_id IS NOT NULL, a.customer_id, NULL)) AS order_arrange_uv,
           sum(IF (a.val_cid = 'page_order_confirm'
                   AND a.val_bid = 'b_drqbsnud'
                   AND d.order_id IS NOT NULL, d.arranged_amt, 0)) AS conversion_sales_amt
    FROM mart_caterb2b.fact_flow_visit_click_kv_day a
    JOIN
      (SELECT m.dt,
              m.customer_id,
              n.region_id,
              n.region_name,
              n.management_city_id,
              n.management_city_name
       FROM mart_caterb2b.dim_caterb2b_customer_his m
       JOIN mart_caterb2b.dim_sm_management_city_info n ON m.city_id = n.city_id
       WHERE m.dt = '$now.datekey') b ON a.customer_id = b.customer_id
    AND a.dt = b.dt
    LEFT JOIN
      (SELECT e.dt,
              e.parent_order_id,
              c.order_id,
              c.arranged_amt
       FROM mart_caterb2b.fact_biz_order_day e
       left JOIN
         (SELECT dt,
                 order_id,
                 sum(arranged_amt) AS arranged_amt
          FROM mart_caterb2b.mid_deal_order_item_withpop
          WHERE dt = '$now.datekey'
            AND is_arranged = 1
            AND channel_id = 1001
            AND order_type NOT IN (3,
                                   4)-- 排除补货换货

            AND spu_type != 5 -- 排除包装物

          GROUP BY dt,
                   order_id) c ON e.dt = c.dt
       AND e.order_id = c.order_id
       WHERE e.dt = '$now.datekey') d ON a.order_id = d.parent_order_id
    AND a.dt = d.dt
    WHERE a.dt = '$now.datekey'
    GROUP BY b.region_id,
             b.region_name,
             b.management_city_id,
             b.management_city_name,
             a.device_type_id
    GROUPING SETS ((b.region_id, b.region_name, b.management_city_id, b.management_city_name, a.device_type_id),
               (b.region_id, b.region_name, a.device_type_id),
               (b.region_id, b.region_name, b.management_city_id, b.management_city_name),
               (b.region_id, b.region_name),
               (a.device_type_id),
               ())
```



但是并没有改善

说明a表与b表关联时候，由于能用order_id关联上是少部分的数据（下单数据），而不能关联上的还是被分配到两个节点上处理。

所以只能将数据拆分处理，sql如下：

```sql
WITH flow_log_info AS
  (SELECT a.dt,
          b.region_id,
          b.region_name,
          b.management_city_id,
          b.management_city_name,
          a.device_type_id,
          a.customer_id,
          a.event_type,
          a.val_cid,
          a.val_bid,
          a.order_id
   FROM mart_caterb2b.fact_flow_visit_click_kv_day a
   JOIN
     (SELECT m.dt,
             m.customer_id,
             n.region_id,
             n.region_name,
             n.management_city_id,
             n.management_city_name
      FROM mart_caterb2b.dim_caterb2b_customer_his m
      JOIN mart_caterb2b.dim_sm_management_city_info n ON m.city_id = n.city_id
      WHERE m.dt = '$now.datekey') b ON a.customer_id = b.customer_id
   AND a.dt = b.dt
   WHERE a.dt = '$now.datekey')

INSERT OVERWRITE TABLE `${target.table}` PARTITION (dt)
SELECT coalesce(e.region_id, '-1') AS region_id,
       coalesce(e.region_name, '全部') AS region_name,
       coalesce(e.management_city_id, '-1') AS management_city_id,
       coalesce(e.management_city_name, '全部') AS management_city_name,
       coalesce(e.device_type_id, '-1') AS device_type_id,
       e.dau,
       e.home_expose_uv,
       e.home_click_uv,
       e.add_cart_uv,
       e.order_uv,
       e.order_arrange_uv,
       e.conversion_sales_amt,
       if(e.home_expose_uv = 0, 0, 1 - (e.home_click_uv / e.home_expose_uv)) AS bounce_rate,
       if(e.dau = 0, 0, e.order_arrange_uv / e.dau) AS dau_arrange_rate,
       if(e.dau = 0, 0, e.conversion_sales_amt / e.dau) AS sales_amt_per_uv,
       '$now.datekey' AS dt
FROM
  (SELECT c.region_id,
          c.region_name,
          c.management_city_id,
          c.management_city_name,
          c.device_type_id,
          sum(c.dau) AS dau,
          sum(c.home_expose_uv) AS home_expose_uv,
          sum(c.home_click_uv) AS home_click_uv,
          sum(c.add_cart_uv) AS add_cart_uv,
          sum(c.order_uv) AS order_uv,
          sum(c.order_arrange_uv) AS order_arrange_uv,
          sum(c.conversion_sales_amt) AS conversion_sales_amt
   FROM
     (SELECT a.region_id,
             a.region_name,
             a.management_city_id,
             a.management_city_name,
             a.device_type_id,
             count(DISTINCT a.customer_id) AS dau,
             count(DISTINCT IF (a.event_type = 'view'
                                AND a.val_cid = 'page_csu_list', a.customer_id, NULL)) AS home_expose_uv,
             count(DISTINCT IF (a.event_type = 'click'
                                AND a.val_cid = 'page_csu_list', a.customer_id, NULL)) AS home_click_uv,
             count(DISTINCT IF (a.val_bid = 'b_h856xuac', a.customer_id, NULL)) AS add_cart_uv,
             count(DISTINCT IF (a.val_cid = 'page_order_confirm'
                                AND a.val_bid = 'b_drqbsnud', a.customer_id, NULL)) AS order_uv,
             0 AS order_arrange_uv,
             0 AS conversion_sales_amt
      FROM flow_log_info a
      GROUP BY a.region_id,
               a.region_name,
               a.management_city_id,
               a.management_city_name,
               a.device_type_id
      GROUPING SETS ((a.region_id, a.region_name, a.management_city_id, a.management_city_name, a.device_type_id),
               (a.region_id, a.region_name, a.device_type_id),
               (a.region_id, a.region_name, a.management_city_id, a.management_city_name),
               (a.region_id, a.region_name),
               (a.device_type_id),
               ())

      UNION ALL

      SELECT a.region_id,
           a.region_name,
           a.management_city_id,
           a.management_city_name,
           a.device_type_id,
           0 AS dau,
           0 AS home_expose_uv,
           0 AS home_click_uv,
           0 AS add_cart_uv,
           0 AS order_uv,
           count(DISTINCT IF (a.val_cid = 'page_order_confirm'
                              AND a.val_bid = 'b_drqbsnud'
                              AND d.order_id IS NOT NULL, a.customer_id, NULL)) AS order_arrange_uv,
           sum(IF (a.val_cid = 'page_order_confirm'
                   AND a.val_bid = 'b_drqbsnud'
                   AND d.order_id IS NOT NULL, d.arranged_amt, 0)) AS conversion_sales_amt
      FROM flow_log_info a
      JOIN
        (SELECT e.dt,
                e.parent_order_id,
                c.order_id,
                c.arranged_amt
         FROM mart_caterb2b.fact_biz_order_day e
         LEFT JOIN
           (SELECT dt,
                   order_id,
                   sum(arranged_amt) AS arranged_amt
            FROM mart_caterb2b.mid_deal_order_item_withpop
            WHERE dt = '$now.datekey'
              AND is_arranged = 1
              AND channel_id = 1001
              AND order_type NOT IN (3,
                                     4)-- 排除补货换货

              AND spu_type != 5 -- 排除包装物

            GROUP BY dt,
                     order_id) c ON e.dt = c.dt
         AND e.order_id = c.order_id
         WHERE e.dt = '$now.datekey') d ON a.order_id = d.parent_order_id
      AND a.dt = d.dt
      GROUP BY a.region_id,
               a.region_name,
               a.management_city_id,
               a.management_city_name,
               a.device_type_id
      GROUPING SETS ((a.region_id, a.region_name, a.management_city_id, a.management_city_name, a.device_type_id),
               (a.region_id, a.region_name, a.device_type_id),
               (a.region_id, a.region_name, a.management_city_id, a.management_city_name),
               (a.region_id, a.region_name),
               (a.device_type_id),
               ()) )c
   GROUP BY c.region_id,
            c.region_name,
            c.management_city_id,
            c.management_city_name,
            c.device_type_id) e
```





## 总结

当用join关联时，一定要注意数据的量级分布，包括能关联上的和不能关联上的，提前计算一下！

## 原理探究





