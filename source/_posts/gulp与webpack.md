---
title: gulp与webpack
date: 2018-09-19 16:25:44
tags: [技术, gulp, webpack]
---
自从前端在项目开发中占比越发重要，跳脱了静态页面以及动画实现的单一工作之后，前端项目工程化的口号被喊响，加之三大框架把我们命令式的开发转为声明式的开发，使得前端工程师不再过度关注数据变更之后视图要如何变动的问题，前端项目则变得越来越大，越来越难以维护。这时候，自动构建工具出现在人们眼中。

前端构建工具经历了漫长的衍生期，从Grunt到gulp，再到webpack，每一种工具在其统治时期内都显得相当耀眼。

就目前来看，在前端面试过程中，gulp与webpack总会被提及，尤其是后者，在模块化越发流行的今天，更是一时间风头无二。似乎问“**有没有自己配置过webpack**”成了一个必须项，并且“**有过了解，可以修改配置，但没有自己实现过**”的回答总显得那么尴尬……

<!--more-->

### 简单聊聊gulp

>gulp 致力于 自动化和优化 你的工作流，它是一个自动化你开发工作中 痛苦又耗时任务 的工具包。

介绍gulp的技术或者教程文章有很多，无论是简书或者是掘金还是CSDN，但是无论在哪，对gulp的描述中，都少不了一个字“流”，而在其核心API `src`中有一个重要的函数，叫`pipe`，也就是管道的意思，也从侧面反映了gulp“流”的特性。

“流”就是 水流 流水的那种形象，常见的 “流水作业”就是这样一个意思。gulp的目的在于把整个项目的工作流程用“流”的方式管理起来，使得开发更具有效率性。实际上，这个“流”除了指代形象的描述以外，还指代的是“**数据流**”，gulp就是基于node开发的，但是其明没有直接使用nodeJS的“fs”模块的IO和流，而是封装了一层“vinil”，使得整个流的过程更加的简洁明了。

这个，没有深入了解，有兴趣可以查阅一下相关资料。

当然，gulp 是如何接过 grunt 的大旗的呢？我没有使用过grunt，所以也不好发表具体的意见，但是我在其官网上找到一个PPT，其中一页是这样的：

![grunt的不好的地方](https://ws1.sinaimg.cn/large/0064OUUqly1fvdsa6enofj30xn0icwi3.jpg)

总结起来大概就是 grunt 的设计太过于繁琐，想包揽一切任务，结果使自己变得混乱不堪。同时，在最后一行小字，声明了 gulp 的设计哲学：它应该只操控文件即可，而让其他的库去处理其余部分。也就是，**精准定位，各司其职**。

不得不说，这种理念堪称经典，也正是由于这种思想，诞生了方便又好用的 gulp 。

#### gulp核心API

如果打开gulp的官方文档就会发现，其API文档只有一页，含有四个API的使用说明。这四个API就是gulp最核心也是全部的API。

- src 负责输出
- dest 能写文件，重新输出
- task 就是任务，最核心的api，在task函数里，可以写甚多操作。
- watch 监视文件，并且可以在文件发生改动时候做一些事情

通过这四个API，我们几乎可以完成项目中所有复杂的构建，只要把任务写清，什么文件被怎样处理然后输出到哪里即可。

### 重量级的webpack

> Webpack 是当下最热门的前端资源模块化 管理和打包 工具。它可以将许多松散的模块按照依赖和规则打包成符合生产环境部署的前端资源。还可以将按需加载的模块进行代码分割，等到实际需要的时候再异步加载。

随着前端的不断发展，模块化的方案越发流行。AngularJS退出历史舞台，React隐隐有一家独大之趋势，而Vue后来居上，Angular短短几年，也从版本2发布到了V6.1，且也开启了组件化之旅。在这样的大趋势下，gulp显得有些力不从心，通过“流”的思想去处理各种模块和组件就尤其怪异。在这种情形下，Webpack揽过模块化打包大旗，手提宝剑，翻身上马，开始进阶杀敌之路，登上历史舞台，最终一统天下。

`Webpack`功能非常强大，而我们使用的时候，几乎不关心内部怎么处理，怎么实现，只需要按需配置文件即可，可以说，`Webpack`本身就是一个功能强悍的模块。

因为他的功能太强大，所以很多时候，知其然不知其所以然，毕竟`，webpack`文档相比于`gulp`，就多了不只有几倍那么简单。

不过幸好，`webpack`的设计理念是很清晰且易于理解的，如果我们理解了其核心的概念，那么后续文档和实现都会相对容易一些。

#### 四个概念

 - **入口** 入口起点(entry point)指示 `webpack` 应该使用哪个模块，来作为构建其内部依赖图的开始。
 - **出口** output属性告诉 `webpack` 在哪里输出它所创建的 bundles，以及如何命名这些文件
 - **loader** `loader` 让 `webpack` 能够去处理那些非 `JavaScript` 文件（`webpack` 自身只理解 `JavaScript`）。
 - **插件** `loader` 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。从打包优化和压缩，一直到重新定义环境中的变量
 
### 看看不同点

其实从`webpack`前两个概念来看，就能明显感受到`webpack`处理文件与`gulp`的明显不同，`webpack`让我们指定一个入口文件，以及最后的输出地址和命名，中间的过程，我们是不需要关心的。而`gulp`则只提供处理文件的API，至于中间的具体处理过程，需要我们亲力亲为。

而至于为什么`webpack`是现在最火的模块化打包方案，原因还是出现在模块化这三个字身上。`gulp`的问题就是不能友好的处理模块化，因为他是基于流的,你想把什么资源引入打包到哪里只能自己手写任务。所以对于`Vue`这种，就显得有些力不从心。而`在webpack`的思想里，一切皆模块。只需要从主文件开始，自动把所依赖的模块引入进来，然后输出一个或者多个文件。

`gulp` 只是对静态资源做流式处理，处理之后并未做有效的优化整合。

常见的就是 A文件引入了B文件，C文件也引入了B文件，主文件引入了A和C，在`gulp`里，写任务处理，就会对B文件引入两次，但是`webpack`就是会引入一次。

两者相比，单纯从使用来说，差不太多，`gulp`的四个api很简单，可以基于他们实现很多打包功能。而`webpack`则是基于各种不同的配置，通过那四个概念完成打包构建。

但是从原理理解上来说，`gulp`明显就更加直观直接一些，你写的任务就是他要做的事情，他不会做任务以外的事情。但是`webpack`明显自身内封装了很多东西，使得它自身更强大，但是理解起来，就显得更复杂一些。

### 小试牛刀

我个人对webpack目前就是局限于看文档改配置文件的程度，所以被面试问到这个问题的时候，就时常感觉尴尬~

所以，一定要自己亲自尝试一下，自己配置一个`vue`打包构建的`webpack`配置文件。

#### 最简单的配置走起

第一步，当然是要初始化项目以及安装webpack

```
npm install webpack webpack-cli --save-dev
```

接下来新建文件，设置目录。依赖关系如下：

![引入依赖关系](https://ws1.sinaimg.cn/large/0064OUUqly1fvev2nbyuqj30840613ym.jpg)

`index.html` 引入了 `index.js`

index.js 文件内容

```javaScript
import _ from 'lodash';
import bar from './module/bar';
import test from './module/moduleTest1';
function component() {
  var element = document.createElement('div');
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
  return element;
}
bar();
test();
document.body.appendChild(component());
```

`index.js` 分别引入了由`npm`安装的`lodash`以及自定义实现的`bar``和moduleTest1`

bar.js
```javaScript
export default function bar() {
  console.log('this is module ____ bar');
  return true;
}
```

moduleTest1.js
```javaScript
import test2 from './moduleTest2.js';
export default function test() {
  test2();
  console.log('this is test 1 function');
}
```

`moduleTest1`又引入了`moduleTest2`

moduleTest2.js
```javaScript
export default function test2() {
  console.log('this is test 2 function');
}
```



新建一个 webpack.config.js 文件，然后开始写入配置：

```javaScript
const path = require('path');

module.exports = {
  entry: './src/index.js',  // 入口文件
  output: { // 输出配置
    path: path.resolve(__dirname, 'dist'),  // 定义输出路径
    filename: 'bundle.js'   // 定义输出文件名
  }
};
```

控制台输入：`npx webpack` 自动运行打包，然后再dist文件夹下生成`bundle.js`，然后把index.html复制进去，手动改成引入 bundle 即可。

运行结果：

![控制台输出](https://ws1.sinaimg.cn/large/0064OUUqly1fvevboc4jhj30lz094gn3.jpg)

![运行结果](https://ws1.sinaimg.cn/large/0064OUUqly1fvevcsj9hij30yx0e9gmx.jpg)

可以看到，在配置文件中，只规定了入口文件以及输出文件的基本信息，然后在打包构建的过程中，`webpack`**自动**将所有的相关的模块都自动打包到了 bundle.js 里。也就是说，无论我是随意的删减其中的模块还是增加，都不会影响配置文件的写法，也不会影响打包的成功，因为配置本身并不关心如何处理过程，而是在处理的过程中动态的解析内容，进行导入的。

#### 试试loader

webpack本身只认识JS，所以，如果我们想要它能正确的处理CSS之类的其他文件，就需要用到loader。

```
npm install --save-dev style-loader css-loader
```

接下来在index.css里加一些样式规则：
```css
body {
  color: red;
}
```

然后，在index.js里引入进去

```javaScript
import './index.css';
```

改造一下 config

```javaScript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 匹配规则
        use: ['style-loader', 'css-loader'] // 使用的loader
      }
    ]
  }
};
```

然后就可以在控制台输入构建命令（可以在package.json的scripts属性中加入配置添加脚本命令：`"build:": "webpack"`，然后就可以使用`npm run build`），浏览器查看效果了。

![样式表](https://ws1.sinaimg.cn/large/0064OUUqly1fvevyjhp16j309f030gle.jpg)

另外，其他所有文件类型如图片、字体、资源等，都可以同过这种方式去处理。

#### 加上Vue实现的配置

重新清理项目目录，如下构建

![重构](https://ws1.sinaimg.cn/large/0064OUUqly1fvexcjvgvaj30bs04ut8p.jpg)

改一下配置文件，加入vue
```javaScript
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  plugins: [new VueLoaderPlugin()],
  module: {
    rules: [
      {
        test: /\.css$/, // 匹配规则
        use: ['style-loader', 'css-loader'] // 使用的loader
      },
      {
        test: /\.vue$/, //VUE文件
        loader: 'vue-loader' //加载器
      },
      {
        test: /\.js$/,
        loader: 'babel-loader', //es6转换，所有的es6文件加载及转换
        exclude: /node_modules/ //排除这个目录
      }
    ]
  }
};

```

其他文件就比较常规了，直接构建即可：
![最终构建](https://ws1.sinaimg.cn/large/0064OUUqly1fvexe8hw6zj311s0buq5a.jpg)