---
    title: git操作
    categories: tools
    tags:
    creator: cjq
    create_time: 2021/01/06

---



## 编辑

删除工作区某个文件，或某个目录的改动

```shell
git checkout <file-name>
git checkout <path-name>
```





## 查看

[#如何使用git比较两次commit之间的差异文件](https://blog.csdn.net/u012830148/article/details/77497240)

```shell
git diff hash1 hash2
hash2是后提交的commit
即hash1是被对比commit，diff结果是hash2在hash1上的改动
```





查看本地分支关联的远端分支

```shell
git branch -vv
```

