---
title: Ubuntu安装GIt
date: 2018-07-28 21:44:25
tags: [ubuntu, git, 技术]
---

## 安装

ubuntu 下安装 git 非常方便，直接用自带的命令安装即可

```
sudo apt-get install git
```

安装成功后，即可使用`git`查看相关命令

<!--more-->

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqsfmca4mfj30jf0i60ue.jpg)

## 设置账号与邮箱

```
// 你的github账号信息
git config --global user.name "yourUserName"
git config --global user.email "yourEmailAddress"
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqsfnwyamkj30k900x0sn.jpg)

## 设置生成秘钥，绑定到账号

```
ssh-keygen -t rsa -C "yourEmailAddress"
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqsfqeeb52j30iv048mxj.jpg)

生成秘钥之后，进入提示的目录，把公钥（id_rsa.pub）下载到本地,用记事本打开，复制里面的内容。

进入 https://github.com/settings/keys ，选择添加 Key

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqsfs6m427j30o403a0ss.jpg)

然后把复制好的公钥内容粘贴到内容区，保存。

此时，可以再 ubuntu 查看连接：

```
ssh -T git@github.com
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqsftcyivpj30lq012a9y.jpg)
