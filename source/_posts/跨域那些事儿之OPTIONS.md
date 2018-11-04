---
title: 跨域那些事儿之OPTIONS
date: 2018-10-15 18:25:33
tags: [技术, 跨域, 预检请求, CORS, OPTIONS]
---

10.16 百度面试时候，二面被问了一些关于预检请求 OPTIONS 的问题，发现自己对于跨域的理解还是处于很**肤浅**的层面：

只是知道受**浏览器同源策略**限制，当请求与当前**协议**、**域名**、**端口**出现不一致情况时，浏览器就会在真正的业务请求发送之前，先发送一个**OPTIONS**请求来获取允许跨域的方法。

<!--more-->

这两天，把 MDN 关于预检请求的一些资料又仔细看了一遍，算是重新了解了一下 CORS 以及 OPTIONS。

首先，MDN 上关于跨域请求的发起解释：

> 当一个资源从与该资源本身所在的服务器不同的域或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。

同时，关于 script/img/audio/video 等含有 src 属性的标签，获取非同源资源时候，发起的也是跨域请求。

但是，并非是跨域请求就会被拦截，不被拦截的都不是跨域，。只要有任一项与同源策略不相符，那么就是在跨域，之所以出现拦截与否的条件，只是因为浏览器在处理跨域问题的不同对待。

![跨域表现](https://ws1.sinaimg.cn/large/0064OUUqly1fw8yzsi1nuj30ho0bk76k.jpg)

那么，什么时候跨域请求会被拦截？

> 出于安全原因，浏览器限制从脚本内发起的跨源 HTTP 请求。例如，XMLHttpRequest 和 Fetch API 遵循同源策略。 这意味着使用这些 API 的 Web 应用程序只能从加载应用程序的同一个域请求 HTTP 资源，除非使用 CORS 头文件

所谓 CORS，就是指的**跨域资源共享**，是现今很常见的跨域手段。主要操作就是，后台通过设置`Access-Control-Allow-Origin`来设置允许跨域访问的域名，如设置通配符`*`，则允许任意域名访问。

跨域资源共享标准（ cross-origin sharing standard ）允许在下列场景中使用跨域 HTTP 请求：

- 由 XMLHttpRequest 或 Fetch 发起的跨域 HTTP 请求。
- Web 字体 (CSS 中通过 @font-face 使用跨域字体资源), 因此，网站就可以发布 TrueType 字体资源，并只允许已授权网站进行跨站调用。
- WebGL 贴图
- 使用 drawImage 将 Images/video 画面绘制到 canvas
- 样式表（使用 CSSOM）
- Scripts (未处理的异常)

#### 简单请求

某些请求不会触发 CORS 预检请求。这样的请求被称为“简单请求”，但是，该术语并不属于 Fetch （其中定义了 CORS）规范。

所谓**简单请求**是指：

- 请求方法是 `GET`、`HEAD`、`POST`之一
- 请求头只有`对CORS安全的首部字段`:
  - Accept
  - Accept-Language
  - Content-Language
  - Content-Type (值只能是`text/plain`/`multipart/form-data`/`application/x-www-form-urlencoded`)
  - DPR
  - Downlink
  - Save-Data
  - Viewport-Width
  - Width
- 请求中的任意 XMLHttpRequestUpload 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问。
- 请求中没有使用 ReadableStream 对象

**注**：这些跨域请求与浏览器发出的其他跨域请求并无二致。如果服务器未返回正确的响应首部，则请求方仍旧不会收到任何数据。

![检视请求](https://ws1.sinaimg.cn/large/0064OUUqly1fw91rs187yj30ix08i0tx.jpg)

#### 预检请求

需要使用预检请求的必须在真正的业务请求发出之前，发送**OPTIONS**请求以获知服务器是否允许该实际请求。

预检请求报文中的`Access-Control-Request-Method` 首部字段告知服务器实际请求所使用的 HTTP 方法；`Access-Control-Request-Headers` 首部字段告知服务器实际请求所携带的自定义首部字段。服务器基于从预检请求获得的信息来判断，是否接受接下来的实际请求。

请求头示例：

```
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
# 发起请求域
Origin: http://foo.example
# 实际请求方法是 POST
Access-Control-Request-Method: POST
# 自定义请求头 X-PINGOTHER
# 使用了 Content-Type 的其他值 比如 application/json
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

服务器所返回的 `Access-Control-Allow-Methods` 字段将所有允许的请求方法告知浏览器。

> `Access-Control-Allow-Methods`首部字段与 `Allow` 类似，但只能用于涉及到 CORS 的场景中

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
# 允许 foo.example 跨域
Access-Control-Allow-Origin: http://foo.example
# 允许 POST GET OPTIONS 请求方法
Access-Control-Allow-Methods: POST, GET, OPTIONS
# 允许的额外头部字段
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
# 允许的缓存时间
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

#### 优化 Web 性能之减少预检请求

从某一角度讲，跨域请求是不可避免的，但是，如果跨域请求有 1000 个，那么是不是要有对应的**1000 个预检请求**。

百度二面时，面试官特意问了**如何尽肯能减少 OPTIONS 请求以优化性能**，这个由于对于预检请求的认知甚浅，所以没说上来。

而现在，这个问题则可以很清晰的来分析一下：

首先，跨域的情况是不能避免的，如果避免了，也就不会有预检请求了，又谈何减少呢？根本是一个都没有。

其次，如果跨域不能避免，那么`OPTIONS`请求可以避免吗？

从之前的“简单请求”可以知道，当一个跨域请求符合浏览器认为的安全跨域请求情况下，虽然跨域仍然存在，但是 OPTIONS 却不会发送了。所以，可以试着把“复杂请求”转化为“简单请求”？

答案是，**No**。

首先就是方法，现在开发都在遵循的`Restful Api`规范，就是明确制定了 `PUT` 和 `DELETE` 方法的必要性，如果省掉了这两个方法，会增加很多交流上的成本问题。

再其次就是 `Headers`，现在我们大多在用 `JSON`作为前后端交互的数据格式，而`application/json`则不在“简单请求”的范围内。不仅如此，为了安全问题，我们在请求时候，往往会在`header`里增加一个自定义字段`token` 用于用户身份校验。这个字段也是不能省略的。

从方法到 Header 都不允许我们把业务请求接口变成“简单请求”，所以，应该有的业务请求接口的`OPTIONS`是不能被省略的。

但是，在实际使用中，web 应用经常会遇到**刷新操作**，这时候，同样的接口，也要发送`OPTIONS`请求，那么这些**重复的跨域请求**是否可以避免`OPTIONS`预检请求呢？

答案是， **Yes**

可以看到，在预检请求的响应头部有一个字段是`Access-Control-Max-Age`，这个请求的有效时间，按上面示例来说`86400`表示的是在 24 小时内，同一请求无需再发送预检请求。善用这个响应头，则可以为重复请求减少`OPTIONS`的发送。

另外需要注意的是，同一请求指的是整个请求从 URL 到方法到 Header 到参数到数据，都要完全一致，才会认为这是同一请求，所以，即便只是增加了一个参数或者修改了一个参数值，仍旧不能避免`OPTIONS`的发送。

且，`Access-Control-Max-Age`的值也并非是可以设置无限大的，而是由浏览器决定，浏览器自身维护了一个最大有效时间，如果该首部字段的值超过了最大有效时间，将不会生效。

_——文章内容大多来自于 MDN 文档——_
