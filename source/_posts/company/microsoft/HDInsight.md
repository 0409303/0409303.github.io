---
    title: HDInsight
    categories: microsoft
    tags:
    creator: cjq
    create_time: 2021/12/17




---

#### [guide line](https://docs.microsoft.com/en-us/azure/hdinsight/)



## best practise

#### 新建cluster注意事项：

1. 指定独立的VNet，保证可以做灵活的网络管控
2. 对于节点多的cluster，为了保证Ambari监控性能，最好申请独立的DB，并且提高其等级，机器越多建议提高的等级越多。

