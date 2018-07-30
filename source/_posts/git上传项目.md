---
title: git上传项目
date: 2018-07-28 21:42:16
tags: [git, 技术]
---

## 上传新项目到 git 仓库

1.  先进入项目文件夹）通过命令 git init 把这个目录变成 git 可以管理的仓库

```
git init
```

<!--more-->

2.  把文件添加到版本库中，使用命令 git add .添加到暂存区里面去，不要忘记后面的小数点“.”，意为添加文件夹下的所有文件

```
git add .
```

3.  用命令 git commit 告诉 Git，把文件提交到仓库。引号内为提交说明

```
git commit -m 'first commit'
```

4.  关联到远程库

```
git remote add origin https://xxx/xxx/xx.git
```

5.  把本地库的内容推送到远程，使用 git push 命令，实际上是把当前分支 master 推送到远程。执行此命令后会要求输入用户名、密码，验证通过后即开始上传。

```
git push -u origin master
```

6.  状态查询命令

```
1 git status
```

## Git 常用命令查询表

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqxyumbhlpj31lo14q4qq.jpg)
