---
    title: 多流join
    categories: spark-streaming
    tags:
    creator: cjq
    create_time: 2021/07/06



---



And as with all things stateful in Structured Streaming, checkpointing ensures that you get exactly-once fault-tolerance guarantees.



### 参考资料：

1. [Introducing Stream-Stream Joins in Apache Spark 2.3](https://databricks.com/blog/2018/03/13/introducing-stream-stream-joins-in-apache-spark-2-3.html)
   1. join示例
   2. 设置threshold用于等待延迟数据

