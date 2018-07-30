---
title: CSS盒模型
date: 2018-07-28 21:30:43
tags: [CSS, 盒模型, 技术]
---

# 盒模型定义

所有 HTML 元素可以看作盒子，在 CSS 中，"box model"这一术语是用来设计和布局时使用。

CSS 盒模型本质上是一个盒子，封装周围的 HTML 元素，它包括：边距，边框，填充，和实际内容。

盒模型允许我们在其它元素和周围元素边框之间的空间放置元素。

<!--more-->

# 示例图

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxuxcw8cj30sr0ggqc6.jpg)

- **Margin(外边距)** - 清除边框外的区域，外边距是透明的。
- **Border(边框)** - 围绕在内边距和内容外的边框。
- **Padding(内边距)** - 清除内容周围的区域，内边距是透明的。
- **Content(内容)** - 盒子的内容，显示文本和图像。

可通过设**置 box-sizing:border-box** 清除盒模型定义，width 与 height 均为 content-padding+border

# 示例代码

注意：**CSS 中对 div 设置的 width 与 height 是内容区的宽度与高度，并非整个 div 的宽度与高度**。

divWidth = width + padding-left + padding-right;

divHeight = height + padding-top + padding-bottom;

---

```html
<style>
        #this{
            width: 500px;
            border: 2px solid red;
        }
    </style>
    <div id="main">
    <div id="this">
        所有HTML元素可以看作盒子，在CSS中，"box model"这一术语是用来设计和布局时使用。
        CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：边距，边框，填充，和实际内容。
        盒模型允许我们在其它元素和周围元素边框之间的空间放置元素。
    </div>
    </div>
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxvszsodj314505e769.jpg)

## 内边距

```css
#this {
  width: 500px;
  border: 2px solid red;

  padding-top: 50px;
  padding-left: 100px;
  padding-bottom: 200px;
  padding-right: 150px;
}
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxyytq1ij30ur0czt9p.jpg)

## 外边距

```css
#main {
  border: 2px solid black;
  float: left;
}
#this {
  width: 500px;
  border: 2px solid red;

  padding-top: 50px;
  padding-left: 100px;
  padding-bottom: 200px;
  padding-right: 150px;

  margin-top: 20px;
  margin-left: 20px;
  margin-bottom: 60px;
  margin-right: 50px;
}
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fntxz86cbhj31400cnq4g.jpg)
