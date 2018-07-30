---
title: win10开启IPV6
date: 2018-07-28 21:47:35
tags: [win10, ipv6, 技术]
---

# IPv6 Win10

> **提示**：本文基于 Teredo 隧道开启 IPv6，其他开启 IPv6 的方法暂待完善。

> **特别重要**：找到 网络和共享中心 - 更改适配器设置 - 本地连接（无线网络则找到 WLAN 或蓝牙网络连接）- 属性，把 IPv6 协议 前面的勾去掉，确定。否则会出现一些奇怪的问题。

<!--more-->

## 专业版或企业版

### 命令行：

- Win+R 打开 CMD 或 Windows PowerShell（管理员），输入命令：

```
  // 设置 Teredo 服务器，默认为：win10.ipv6.microsoft.com
  netsh interface teredo set state enterpriseclient server=default

  // 测试 IPv6 连接
  ping -6 ipv6.test-ipv6.com
  ping -6 [2001:470:1:18::125]

  // 重置 IPv6 配置
  netsh interface ipv6 reset
```

- 重启系统
- 通过命令`ipconfig /all` 查看当前网络信息，看到 `Teredo Tunneling Pseudo-Interface` 有以 2001 开头的 IPv6 地址即可。 启动 IE 浏览器，访问 [http://test-ipv6.com](http://test-ipv6.com) 或 [http://ipv6.test-ipv6.com](http://ipv6.test-ipv6.com)，如果选项卡 “**测试项目**” 下面的 “**不使用域名的 IPv6 测试**” 显示成功，则隧道建立成功。_Chrome 浏览器的测试结果可能和 IE 不一样，请注意_
- 如果经过上面操作后仍无法启用 IPv6，可能是 Teredo 服务器无法正常连接，也可能是 Windows 自身配置问题，可尝试以下两种方法：

```
// 第一种：修改 Teredo 服务器为 teredo.remlab.net
netsh interface teredo set state server=teredo.remlab.net

// 第二种：先卸载当前 Teredo 适配器再重新启用
netsh interface Teredo set state disable
netsh interface Teredo set state type=default
ping -6 ipv6.test-ipv6.com
```

### 脚本：

打开 XX-Net 目录，切换到 code\default\gae_proxy\local\ipv6_tunnel，执行 **enable_ipv6.bat**
