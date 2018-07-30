---
title: cordova打包Vue项目
date: 2018-07-28 21:49:41
tags: [cordova, vue, 技术]
---

# 1. 基础环境

- Vue 项目环境：vue-cli+webpack
- cordova 插件：`npm install cordova -g`
- 安卓环境：android SDK

<!--more-->

# 2. 新建 cordova 项目，检测环境

```
# 新建项目
# <文件夹名> <包名> <项目名>
# 包名 项目名 可在 cordova 项目——config.xml中修改
cordova create hello com.example.hello HelloWorld
# 进入项目目录
cd hello
# 添加 安卓 平台
cordova platform add android --save
# 检查你当前平台设置状况:
cordova platform ls
```

在正式打包之前，还需要使用命令检测一下 cordova 是否识别已有的全部环境：

```
cordova requirements
```

![image](https://ws1.sinaimg.cn/large/0064OUUqly1fnttyz5ocjj30z105x0tk.jpg)

# 3. 修改 Vue 项目文件

### 1. /config/index.js

直接把构建方式中的**assetsSubDirectory**和**assetsPublicPath**改为相对路径，开发模式无需更改
否则的话，build 项目后引入的文件路径都是绝对路径，不知道为什么，无法被正常解析
index.html 所引用的 static 目录下文件可正常被获取（之前改两个文件的做法，会导致 static 目录下文件打包后位置与 index.html 引用位置不一致，导致无法使用）
cordova 打包时，也不会出现空白页面
![](https://ws1.sinaimg.cn/large/0064OUUqly1fnttzvo2crj30rb0ac3zw.jpg)

# 4. 构建-->打包

### 1. 在 Vue 项目中：`npm run build`

### 2. 把打包好的 dist 文件夹下全部文件复制到 cordova 项目 www 文件夹下

### 3. 运行 `cordova build android`

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntu09ouk3j30tk035q32.jpg)
