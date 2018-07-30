---
title: cordova权限获取
date: 2018-07-28 21:40:15
tags: [cordova, 权限, 技术]
---

widget 标签加入安卓访问 schemas

```xml
<widget id="com.example.hello" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:android="http://schemas.android.com/apk/res/android" xmlns:cdv="http://cordova.apache.org/ns/1.0">
```

<!--more-->

在 android platform 中加入权限列表

```xml
<platform name="android">
        <allow-intent href="market:*" />
        <icon src="res/icon/android/logo.png" />
        <splash src="res/screen/android/firstView.png" />
        <config-file parent="/*" target="AndroidManifest.xml">
            <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
            <uses-permission android:name="android.permission.CAMERA" />
            <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
        </config-file>
</platform>
```

## 内置 webView

cordova-android-support-gradle-release
cordova-plugin-crosswalk-webview
