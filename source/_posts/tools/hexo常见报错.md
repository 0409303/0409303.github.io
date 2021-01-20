---
    title: hexo常见报错
    categories: tools
    tags:
    creator: cjq
    create_time: 2021/01/20

---



### bad indentation of a mapping entry

1. title中缩进不正常，均要用tab缩进



### warning: adding embedded git repository: themes/next

```
warning: adding embedded git repository: themes/next
hint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
hint: If you meant to add a submodule, use:
hint:
hint: 	git submodule add <url> themes/next
hint:
hint: If you added this path by mistake, you can remove it from the
hint: index with:
hint:
hint: 	git rm --cached themes/next
hint:
hint: See "git help submodule" for more information.
```

[git仓库包含子仓库时，add报错的解决办法](https://cloud.tencent.com/developer/article/1583762)

