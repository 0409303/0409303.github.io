---
    title: git设置
    categories: git
    tags:
    creator: cjq
    create_time: 2021/02/10


---



## .gitconfig

git 添加别名的方式,打开~/.gitconfig文件在其末尾添加：
在命令行输入以下命令：
1、进到根目录
cd ~/
2、打开.gitconfig,
vi ~/.gitconfig

```txt
[alias]
	a = add
	b = branch
	c = commit
	d = diff
	l = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset | %C(bold)%an' --abbrev-commit --date=relative
	r = reset
	aa = add .
	ba = branch -a
	ca = commit -a
	cc = commit -a -m
	cl = clone
	cm = commit -m
	co = checkout
	cp = cherry-pick
	nb = checkout -b
	pl = pull
	ps = push origin master
	st = status

[user]
	name = jingqicao
	email = cjqzhuce@126.com

[diff]
	tool = default-difftool
[difftool "default-difftool"]
	cmd = code --wait --diff $LOCAL $REMOTE
[difftool]
	prompt = false
```

