---
title: Egg入坑
date: 2018-07-28 21:19:30
tags: [Node, Egg, 技术]
---

# 基础测试

### 创建项目

```
$ npm i egg-init -g
$ egg-init egg-example --type=simple
$ cd egg-example
$ npm i
```

<!--more-->

### 编写测试 router

```javaScript
// app/route.js
/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
  const { router, controller } = app;
  router.get('/', controller.home.index);
  // 添加一个API测试路由，分发到controller/test.js 中的index方法
  router.get('/api/v1.0/test', controller.test.index);
};
```

### 编写调用中间件

```javaScript
// app/middleware/reqIntercept.js
/**
 * @author : 忍把浮名换此生
 * @E-mail : 2295161275@qq.com
 * @create : 2018/2/7
 * @description : 中间件测试，简单拦截与获取响应
 */
module.exports = () => {
    return async function (ctx, next) {
        console.log('进入resIntercept中间件', new Date().getTime());
        const query = ctx.query;
        if (query.name !== 'EGG') {
            ctx.body = await ctx.service.handleRes.generraterRes(999);
            ctx.status = 201;
        }
        await next();
        console.log('出去resIntercept中间件', ctx.body);

    };
};
```

### 注册中间件

```javaScript
'use strict';

module.exports = appInfo => {
  const config = exports = {};

  // use for cookie sign key, should change to your own and keep security
  config.keys = appInfo.name + '_1517973199742_1216';
  config.reqIntercept = {
      isRequire: true
  };

  // 跨域插件，需要先在 plugin.js中声明
  config.cors = {
      origin: '*',
      allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH'
  };

  // add your config here
  // 配置中间件
  config.middleware = ['reqIntercept'];

  return config;
};
```

### 编写 Service

```javaScript
// app/service/test.js
/**
 * @author : 忍把浮名换此生
 * @E-mail : 2295161275@qq.com
 * @create : 2018/2/7
 * @description : Service 测试，直接返回一条固定JSON
 */
const Service = require('egg').Service;

class TestService extends Service {
    async find() {
        const user = {
            id: 'name',
            name: 'hello',
            text: 'this is my first egg API'
        };
        console.log('调用到业务service--->', new Date());
        return user;
    }
}

module.exports = TestService;
```

### 编写 controller

```javaScript
// app/controller/test.js
/**
 * @author : 忍把浮名换此生
 * @E-mail : 2295161275@qq.com
 * @create : 2018/2/7
 * @description : controller 测试，调用Service生成响应数据
 */
const Controller = require('egg').Controller;
class getController extends Controller {
    async index() {
        console.log('调用到controller--->', new Date());
        let errno = 0;
        let res = {};
        try {
            // 调用 Service 进行业务处理
            res = await this.ctx.service.test.find();
        } catch (err) {
            console.error('err->', err);
            errno = 500;
        }
        res = await this.ctx.service.handleRes.generraterRes(errno, res);
        // 设置响应内容和响应状态码
        this.ctx.body = res;
        this.ctx.status = 201;
        console.log('controller里生成RES---->', new Date());
    }
}
module.exports = getController;
```

### 调用结果

![](https://ws1.sinaimg.cn/large/0064OUUqly1fo86lq4pfij30t206b0u1.jpg)
