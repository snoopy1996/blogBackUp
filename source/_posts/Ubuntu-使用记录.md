---
title: Ubuntu 使用记录
date: 2018-07-28 20:29:55
tags: [Linux, Ubuntu, 技术]
---

<!--more-->

# ! 基本常用命令（常更）

```
# 调出控制台快捷键
Ctrl + Alt + T
# 查看进程
netstat -apt
# 杀死进程
sudo kill -9 PID
# 后台挂起
nohup <code> &
# 对于XShell 上传文件 可使用 rz
sudo apt-get install lrzsz
rz
# 管道命令（过滤）
netstat -atlp | grep node
ps -ef | grep nginx
```

# !! 虚拟机基本常识

1.  虚拟机联网必须依赖主机中的 VM 服务启动并且有 VMnet1 和 VMnet8 虚拟网卡
2.  桥接和 net 模式都可以使虚拟机正常上网
    - 桥接模式： 使得虚拟机成为与主机连接局域网中一台独立的可上网主机，可与主机及其他同一网络下的机器相互通信
    - net 模式：虚拟机把宿主主机当做路由器使用，可以直接正常上网，但不可与主机像正常局域网中的机器那样相互通信

# 1. 设置 root 密码

```
$ sudo passwd
```

# 2. 更换软件源，更新软件

```
//更新软件源列表
$ sudo apt-get update
//更新软件
$ sudo apt-get upgrade
```

# 3. 安装 Node

### 方法 1

命令自带的 apt-get 安装的 node 版本太低，而且只能使用 **nodejs** 命令，而**不能**使用 **node** 。
去官网下载 Linux 版本的压缩包（非源码），复制到 Ubuntu 中。

- 解压压缩包

```
//由于下载的是tar.xz压缩包，首先要安装 xzutils
$ sudo apt-get xzutils
$ tar -xvf node-*-*.tar.xz
$ cd node-*-*
$ ./node -v
$ ……
$ ./npm -v
$ ……
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxnb0wlxj30ke05jq6g.jpg)

- 软连接到全局

```
# 前者路径为node解压路径
ln -s /home/nodeEXE/node-v8.7.0-linux-x64/bin/node /usr/local/bin/node
ln -s /home/nodeEXE/node-v8.7.0-linux-x64/bin/npm /usr/local/bin/npm
```

任何地方都可使用 node 以及 npm 命令

- 对于 npm install -g 全局安装的命令模块，需要设置 PATH 路径才可正常使用

```
echo -e "export PATH=$(npm prefix -g)/bin:$PATH" >> ~/.bashrc && source ~/.bashrc
# 上面的命令中使用 npm prefix -g 获取node安装目录
# root用户才可使用
```

### 方法 2（便捷）

```
# 利用curl把文件下载到本地 | 修改bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
# 安装nodeJs
sudo apt-get install -y nodejs
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fp8v3unfrvj30az02r0sl.jpg)

# 4. nginx

```
sudo apt-get install nginx
// 打开看开配置文件
sudo vim /etc/nginx/sites-available/default
nginx -c /etc/nginx/nginx.conf
// 关闭 nginx
nginx -s stop
// 重读配置文件
nginx -s reload
```
