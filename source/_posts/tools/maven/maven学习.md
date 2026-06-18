---
    title: maven学习
    categories: tools
    tags:
    creator: cjq
    create_time: 2021/01/27


---


[Maven中plugins和pluginManagement的区别](https://www.cnblogs.com/EasonJim/p/6845012.html)

[maven依赖顺序原则](https://www.cnblogs.com/shawWey/p/7417335.html)



### dependencyManagement

1. 这里包名写错不会报错，不会实际尝试去仓库下载，只有在dependencies中定义的才会检查，才会报错
