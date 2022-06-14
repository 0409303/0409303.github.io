---
    title: git基本命令
    categories: git
    tags:
    creator: cjq
    create_time: 2021/01/30

---

## 标准流程

1. 添加暂存区```git add .```（）



## 分支

### 克隆

```
git checkout -b dev(本地分支名称) origin/develop(远程分支名称)

error log:
fatal: 'origin/MTSparkMSNv2' is not a commit and a branch 'users/jingqicao/M2B_V2' cannot be created from it
need run git pull to update.
```



### 删除

[Git 操作——如何删除本地分支和远程分支](https://chinese.freecodecamp.org/news/how-to-delete-a-git-branch-both-locally-and-remotely/)



### 新分支push到远端

```
git push --set-upstream origin users/jingqicao/performance-optimize
// 需要保证本地分支名称与指定的remote分支名称完全相同，否则
error: src refspec users/jingqicao/performance-optimize does not match any
```



### 本地关联远程分支

```
1. push同时关联
git push --set-upstream origin xxx
2. 关联已经存在的
git branch --set-upstream-to=origin/dev
```



## 内容

### 撤销修改

```bash
// 这是一个危险的操作，会撤销指定文件或全部文件的所有本地修改
git checkout -- <file>
git checkout -- .
```



## 提交(commit)

```
git commit -m ""

git commit -am ""
追加commit注释，一次push不会生成多个commit节点

git commit --amend
可以修改最近一次commit的注释
```



### 查看当前commit

```
git log
```

### 对比两个commit（两个commit可以来自不同分支）

```
hash2 在 hash1上的修改
git diff commit-hash1 commit-hash2 --stat

如果想进一步查看test修改了那些地方：
只需将 git diff hash1 hash2 --stat 中的 --stat改为具体的文件即可

如果想将两次提交的差异部分提取成补丁文件
git diff hash1 hash2 filename > patch_name

如果想将多个文件的差异生成到同一patch文件 则：
git diff hash1 hash2 filename1 filename2 > patch_name

使用工具（左右两个commit，对应了difftool的左右窗口）：
git difftool 199b8598f422485e453f1e4e87aaa6176ff48ad0 4b97457217e29355ddeff0aec8562271c06ac6fe --stat  
```

### 对比两个分支

```
git diff users/jingqicao/temp-master master --name-status
可以展示变更文件的全路径

git diff-tree
```

1. [Git 修改已提交的commit注释](https://www.jianshu.com/p/098d85a58bf1)
