

#### 原始sql:

```sql
WITH sku_info AS
  (SELECT a.sku_id,
          d.bu_id,
          if(c.management_type = 3, 'pop', '自营') AS is_3p,
          b.cat1_id,
          b.cat1_name,
          b.cat2_id,
          b.cat2_name,
          max(on_shelf) AS is_onshelf
   FROM
     (SELECT sku_code AS sku_id,
             sales_grid_id,
             on_shelf,
             bu_id
      FROM mart_caterb2b.dim_prod_all_map_csu__grid_his
      WHERE dt = '20210317'
        AND channel_id = '1001'
      UNION ALL SELECT sku_code,
                       sales_grid_id,
                       on_shelf,
                       bu_id
      FROM mart_caterb2b.fact_caterb2b_csu_grid_onshelf_log
      WHERE update_time >='2021-03-18'
        AND update_time <= '2021-03-19'
        AND on_shelf = 1
        AND channel_id = '1001'
      GROUP BY 1,
               2,
               3,
               4) a
   LEFT JOIN
     (SELECT DISTINCT sku_id,
                      cat1_id,
                      cat1_name,
                      cat2_id,
                      cat2_name
      FROM mart_caterb2b.dim_caterb2b_csu_his
      WHERE dt = '20210318'
        AND channel_id = '1001') b ON a.sku_id = b.sku_id
   LEFT JOIN mart_caterb2b.dim_sm_business_entity c ON a.bu_id = c.bu_id
   LEFT JOIN mart_caterb2b.dim_sell_grid d ON a.sales_grid_id = d.sell_grid_id
   GROUP BY 1,
            2,
            3,
            4,
            5,
            6,
            7)
            
            
            
SELECT coalesce(f.bu_id, '-1') AS bu_id,
       coalesce(f.cat1_id, '-1') AS cat1_id,
       coalesce(f.cat1_name, '全部') AS cat1_name,
       coalesce(f.cat2_id, '-1') AS cat2_id,
       coalesce(f.cat2_name, '全部') AS cat2_name,
       coalesce(f.is_3p, '全部') AS is_3p,
       count(DISTINCT if(is_view = 1, f.sku_id, NULL)) AS view_sku_ct,
       count(DISTINCT if(is_view = 1, f.customer_id, NULL)) AS view_uv,
       count(DISTINCT if(is_view = 1, f.sku_id, NULL), if(is_view = 1, f.customer_id, NULL)) AS view_sku_uv,
       count(DISTINCT if(is_intention = 1, f.sku_id, NULL)) AS view_sku_ct,
       count(DISTINCT if(is_intention = 1, f.customer_id, NULL)) AS view_uv,
       count(DISTINCT if(is_intention = 1, f.sku_id, NULL), if(is_intention = 1, f.customer_id, NULL)) AS view_sku_uv
FROM
  (SELECT d.sku_id,
          d.bu_id,
          d.customer_id,
          coalesce(e.cat1_id, '-99') AS cat1_id,
          coalesce(e.cat1_name, '其他') AS cat1_name,
          coalesce(e.cat2_id, '-99') AS cat2_id,
          coalesce(e.cat2_name, '其他') AS cat2_name,
          coalesce(e.is_3p, '其他') AS is_3p,
          d.is_view,
          d.is_intention
   FROM
     (SELECT if(click.event_type = 'view', 1, 0) AS is_view,
             CASE WHEN (click.val_cid = 'page_csu_detail'
                        AND click.val_bid = 'b_htrmpzlb')
                    OR click.val_bid IN ('b_h856xuac',
                           'b_x6m8625m') THEN 1
                  ELSE 0
              END AS is_intention,
              c.sku_id,
              coalesce(cust.bu_id, '-99') AS bu_id,
              click.customer_id
        FROM mart_caterb2b.fact_flow_visit_click_kv_day click
        JOIN mart_caterb2b.dim_caterb2b_csu_his c
          ON click.csu_id = c.csu_id
        LEFT JOIN
          (SELECT DISTINCT customer_id,
                           bu_id
           FROM mart_caterb2b.dim_caterb2b_customer_his
           WHERE dt = '20210225'
             AND business_type = 2
             AND cat_type = 1
             AND channel_id = '1001') cust ON click.customer_id = cust.customer_id
        WHERE click.dt = '20210225'
          AND click.csu_id IS NOT NULL
          AND c.dt = '20210225' )d
  LEFT JOIN sku_info e ON d.sku_id = e.sku_id
  AND d.bu_id = e.bu_id)f
GROUP BY f.bu_id,
         f.cat1_id,
         f.cat1_name,
         f.cat2_id,
         f.cat2_name,
         f.is_3p
GROUPING sets((f.bu_id, f.cat1_id, f.cat1_name, f.cat2_id, f.cat2_name, f.is_3p),
            (f.bu_id, f.cat1_id, f.cat1_name, f.cat2_id, f.cat2_name),
            (f.bu_id, f.cat1_id, f.cat1_name, f.is_3p),
            (f.bu_id, f.cat1_id, f.cat1_name),
            (f.bu_id, f.is_3p),
            (f.bu_id),
            (f.cat1_id, f.cat1_name, f.cat2_id, f.cat2_name, f.is_3p),
            (f.cat1_id, f.cat1_name, f.cat2_id, f.cat2_name),
            (f.cat1_id, f.cat1_name, f.is_3p),
            (f.cat1_id, f.cat1_name),
            (f.is_3p),
            ())
```



#### 添加参数：

SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.dynamic.partition=true;
-- 分区
SET hive.exec.max.dynamic.partitions=1000;
-- 触发spark合并小文件的阈值(1M)
SET spark.sql.mergeSmallFileSize=1048576;
-- spark shuffle默认分区数
SET spark.sql.shuffle.partitions=500;
-- 限制最大申请资源
set spark.dynamicAllocation.enabled=true;
set spark.dynamicAllocation.minExecutors=3;
set spark.dynamicAllocation.maxExecutors=300;
set spark.executor.cores=2;

如果不添加参数，在最后一步，资源急剧下降，导致执行时间很长