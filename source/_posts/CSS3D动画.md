---
title: CSS3D动画
date: 2018-07-28 21:34:04
tags: [CSS, 动画, 技术]
---

#### 1. 把带有 3D 效果元素的父元素设置为舞台。

```css
{-webkit-perspective:4000px;//4000px是舞台距离}
```

#### 2. 如果舞台元素的孙子元素同样需要 3D 效果，则为其子元素设置保留 3D

```css
 {
  transform-style: preserve-3d;
}
```

<!--more-->

#### 3. 定义动画

```css
/*keyframs定义动画*/
/*from……to可以换为动画进程的百分比*/
@keyframes roll {
  from {
    transform: rotate3d(0.5, 0.5, 0.5, 0deg);
  }
  to {
    transform: rotate3d(0.7, 0.7, 0.7, 360deg);
  }
}
```

#### 4. animation 属性使用动画。

```
animation: roll 30s linear infinite;
```

#### 5. 备注（CSS3 相关属性用法详解）：

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxi46t1xj30jr08njru.jpg)
animation-timing-funtion
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxion1p1j30jq0c9t9d.jpg)
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxizvl1oj30i80jaq46.jpg)
![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxjc70okj30jq03nwed.jpg)
