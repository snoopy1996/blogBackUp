---
title: Node爬取《西虹市首富》短评生成词云
date: 2018-07-31 19:35:05
tags: [Node, 爬虫, 词云]
reward: true
---

近期《西虹市首富》3 天破 9 亿的新闻铺天盖地，作为开心麻花又一力作，想必从笑料方面不会让人失望。然后看到掘金小妹发布了朋友圈，一篇爬取西虹市首富猫眼短评数据生成词云的文章，点进去一看，是 python 版的源码。然后忽发奇想，何不用 node 实现一版呢？

<!--more-->

## 准备阶段

在爬取数据之前，当然是要确定爬取的 URL 了……

作为一个天真的人，从容打开了豆瓣影评，然后看到了“遮天蔽日”5w+短评数量，顿时把原文猫眼给扔到一边了。

基本解析了一下页面结构，发现，所有的短评都是 `.short`的文本内容。嗯……，这应该是写过的爬虫里最好解析的了，当然，也没有要其他数据的想法。

![数据](https://ws1.sinaimg.cn/large/0064OUUqly1ftt5rphvbhj319o0gzagm.jpg)

## 开始

### 步骤

爬虫的原理，已经不是什么值得废笔墨去描述的问题了。

1. 获取 URL 列表
2. 访问 URL 内容
3. 解析页面内容
4. 存储数据

### 所需依赖

初始化项目，下载依赖

```javascript
// 加载依赖
const axios = require('axios'); // 访问http
const asyncNpm = require('async'); // 异步并发控制
const fs = require('fs'); // 文件模块
const cheerio = require('cheerio'); // 解析页面所用
```

### 具体代码

```javascript
let index = 0;
/**
 * 收取一页的短评
 * @param {String} url
 * @param {Function} callback
 */
const getComment = async (url, callback) => {
  // 控制并发时间间隔，防止IP 403
  const delay = Math.ceil(Math.random() * 5000 + 2500);
  const res = await axios.get(url);
  const $ = cheerio.load(res.data);
  if ($ && $ !== null) {
    const comments = $('#comments .short');
    for (let i = 0, max = comments.length; i < max; i++) {
      const shortContent = comments.eq(i).text();
      fs.appendFile('comments.txt', shortContent + '\r\n', err => {
        if (err) {
          console.log('写入失败->', err);
          return;
        }
        console.log('写入', ++index);
      });
    }
  }
  setTimeout(() => {
    callback(null, url);
  }, delay);
  return true;
};

const commentCount = 50000; // 定义要获取的短评数量
/**
 * 构造短评页面的url列表
 */
const getUrls = () => {
  let start = 0;
  let urlList = [
    'https://movie.douban.com/subject/27605698/comments?start=0&limit=20&sort=new_score&status=P'
  ];
  while (start < commentCount) {
    urlList.push(
      `https://movie.douban.com/subject/27605698/comments?start=${(start += 20)}&limit=20&sort=new_score&status=P`
    );
  }
  console.log(urlList.length);
  return urlList;
};
const urlList = getUrls();
asyncNpm.mapLimit(
  urlList,
  5,
  (url, callback) => {
    getComment(url, callback);
  },
  function(err, result) {
    console.log('result:');
    console.log(result);
  }
);
```

## next

接下来，命令行敲出 `node index.js`，开心的等待 5W+数据的诞生……

![等待数据](https://ws1.sinaimg.cn/large/0064OUUqly1ftt5zn9x2xj30xq09mjt4.jpg)

**WTF?!!**

好吧 ， too young..

如果你没有登录的话，访问 220 条数据之后，是这样子的…

![403](https://ws1.sinaimg.cn/large/0064OUUqly1ftt62xhmo9j312p0av3zm.jpg)

登录的时候，监控了登录接口，然后把所有的数据转移到 postman 进行登录测试。。。失败。。。

### 转战猫眼

猫眼影评的数据可以通过接口获取 json 对象。

```javascript
// offset是页数
// limit参数可选，上限是15 | 2018/7/31
const testApi = 'http://m.maoyan.com/mmdb/comments/movie/1212592.json?offset=2';
```

整体结构

![结构](https://ws1.sinaimg.cn/large/0064OUUqly1ftt69q3z8xj30by03ya9x.jpg)

具体评论

```json
{
  "approve": 0,
  "approved": false,
  "avatarurl": "",
  "cityName": "天津",
  "content": "真的好看，沈腾老师演的电影永远是良心剧，故事的情节生动爆笑，同时伴随一两个泪点，但很快又沉浸于搞笑的氛围中，最后结尾值得让人深刻思考，金钱和人性到底该如何选择，还有一个意想不到的关系嘻嘻算是小惊喜吧，总之绝对的良心剧十分喜欢，推荐推荐！",
  "filmView": false,
  "id": 1032072628,
  "isMajor": false,
  "juryLevel": 0,
  "majorType": 0,
  "movieId": 1212592,
  "nick": "KYC558009090",
  "nickName": "KYC558009090",
  "oppose": 0,
  "pro": false,
  "reply": 0,
  "score": 5,
  "spoiler": 0,
  "startTime": "2018-07-31 17:27:26",
  "supportComment": true,
  "supportLike": true,
  "sureViewed": 0,
  "tagList": {
    "fixed": [
      {
        "id": 1,
        "name": "好评"
      },
      {
        "id": 4,
        "name": "购票"
      }
    ]
  },
  "time": "2018-07-31 17:27",
  "userId": 883221447,
  "userLevel": 1,
  "videoDuration": 0,
  "vipType": 0
}
```

然后，开始了猫眼影评的爬取之路……

故事当然还是要有波折一些，才能写文章。。所以，很顺利的，我遇到了波折：

猫眼影评每天只提供 1000 页数据，当然，1000 \* 15 也有 15000 条了，足够用。问题是，重复量相当多，去重后：

![影评数据](https://ws1.sinaimg.cn/large/0064OUUqly1ftt6pxv7cnj308x03it8n.jpg)

emmm…… 855 + 220 也有 1000+了，，应该可以就和出个词云了。。

## 分词断句

分词工具用的是 `NodeJieba`. 膜拜~

[NodeJieba github](https://github.com/yanyiwu/nodejieba)

![nodejieba](https://ws1.sinaimg.cn/large/0064OUUqly1ftt6u5f5ssj30qa09omys.jpg)

安装时候依赖于 `node-gyp`， [简书有详细安装步骤教程](https://www.jianshu.com/p/2b831714bbff)

然后 node-gyp build 项目出错了

![项目出错](https://ws1.sinaimg.cn/large/0064OUUqly1ftt9tn2rukj312i05p75z.jpg)

目前的成果只有一个 txt ...

等有时间，再好好研究一下 node-gyp 的环境。。貌似是因为 C++ 编译环境的问题。

## 效果

![效果图](https://ws1.sinaimg.cn/large/0064OUUqly1ftt9tg7n4mj30d50adgo9.jpg)

暂时先用了一个在线的分词 词云工具：[图悦](http://www.picdata.cn/index.php)

> 2018/8/1

被自己的系统坑了。。。

`node-gyp` 需要安装 `windows-build-tools` , 而运行安装命令`npm install --global --production windows-build-tools`的时候, 发现里面会自动启用 `powershell` :

![需要依赖powershell](https://ws1.sinaimg.cn/large/0064OUUqly1ftu1xeasoxj30wh07mq3r.jpg)

然后，由于系统崩溃，前几天临时装的系统

作为一个纯洁的人，当然要装一个纯洁的系统。纯洁到`powershell`不存在, 搜索程序也不存在，通知功能被优化砍掉。。。

另外，连`Windows Management Framework 4.0+` 都没有。（安装`powershell`必须依赖于 WMF4.0+）

没有的话，只能手动安装了->

![手动安装](https://ws1.sinaimg.cn/large/0064OUUqly1ftu20l2efij30wo03p74a.jpg)

然而，微软并没有提供 win10 下的安装包！！！

![安装包列表](https://ws1.sinaimg.cn/large/0064OUUqly1ftu1znqrmnj30ru0iv40j.jpg)
