---
title: win10增加右键打开CMD功能
date: 2018-07-29 16:25:30
tags: [系统, win10, cmd, 技术]
---

自从 win10 全面使用 powershell 代替 cmd 之后，就各种不爽……虽然说 powershell 是 cmd 的超集，cmd 能做的，powershell 都能做，但问题是，当用惯了 cmd 之后，再用 powershell，即便用起来都一样，可你还是感觉不一样~而且，真的会有灵异事件，，环境变量在 powershell 找不到，cmd 则成功执行了~

<!--more-->

再加上，新装的系统，竟然没有带 powershell 的程序，所及 shift+右键快捷打开命令行的功能，可见不可用，就很尴尬了。

所以，简单调研了一下，找到一段代码。

```reg
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\Directory\shell\runas]
@="Open cmd here as Admin"
"HasLUAShield"=""
[HKEY_CLASSES_ROOT\Directory\shell\runas\command]
@="cmd.exe /s /k pushd \"%V\""
[-HKEY_CLASSES_ROOT\Directory\Background\shell\runas]
[HKEY_CLASSES_ROOT\Directory\Background\shell\runas]
@="Open cmd here as Admin"
"HasLUAShield"=""
[HKEY_CLASSES_ROOT\Directory\Background\shell\runas\command]
@="cmd.exe /s /k pushd \"%V\""
[-HKEY_CLASSES_ROOT\Drive\shell\runas]
[HKEY_CLASSES_ROOT\Drive\shell\runas]
@="Open cmd here as Admin"
"HasLUAShield"=""
[HKEY_CLASSES_ROOT\Drive\shell\runas\command]
```

新建文本文档，粘入代码，然后直接改后缀名 `.reg` ，双击运行。

上述代码可以修改注册表，添加右键打开 cmd 功能（以管理员身份）。
