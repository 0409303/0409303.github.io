---
    title: git设置
    categories: git
    tags:
    creator: cjq
    create_time: 2021/02/10


---

[toc]



## 安装步骤

```
1. 生成 SSH 密钥，一路回车即可，默认保存在 ~/.ssh/id_ed25519
ssh-keygen -t ed25519 -C "你的邮箱@example.com"

2. 查看公钥并复制
cat ~/.ssh/id_ed25519.pub

3. 添加到 GitHub：
进入 GitHub → Settings → SSH and GPG keys → New SSH key，粘贴公钥内容保存。

4. 测试连接
ssh -T git@github.com

5. 拉取某个分支
git clone -b 分支名 https://github.com/用户名/仓库名.git
```





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

