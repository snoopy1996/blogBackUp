---
title: http方法及其幂等性
date: 2018-08-30 17:42:55
tags: [技术, HTTP, 幂等性, 缓存]
reward: true
---
基于`HTTP`协议的api是当下最流行的服务提供方式，从根本上解决了前后端分离开发的技术难点。当下的`HTTPS`也仅仅是对`HTTP`的封装，而并非一种新的协议，这一点在之前的[https小记](https://blog.supersy.xyz/2018/08/15/https%E5%B0%8F%E8%AE%B0/)有过阐述。

<!--more-->

### HTTP三个特点

- `HTTP`是无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间
- `HTTP`是媒体独立的：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型
- `HTTP`是无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

![HTTP 工作原理](https://ws1.sinaimg.cn/large/0064OUUqly1furt5ose8uj309p05tmx4.jpg) 

### HTTP请求方法

根据`http`标准，请求可以使用多种请求方法。

HTTP1.0 定义了三种： GET， POST， HEAD;

HTTP1.1 新增了五种请求方法：OPTIONS， PUT， DELETE， TRACE 和 CONNECT。

其中，GET和POST最为常见，其次就是基于restful API编程的情况下，PUT和DELETE也有很大的用处，而跨域情况下 OPTIONS 则也很常见。

### 幂等性

**幂等性**又被叫做状态统一性，是HTTP编程中的一个重要特质，定义为：

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.

也就是，同一个HTTP请求在一次和多次请求情况下对服务器上这个资源应该有相同的副作用。

例如请求 `DELETE http://server.com/auth/user/1` 表示删除服务器上认证用户表中id为1的用户，执行第一次的时候，用户被删除，结果是 服务器上不存在id为1的用户，第二次执行，虽然没有删除任何用户（id为1的用户已被删除），但是结果是一样的：服务器上不存在id为1的用户，当执行第三次、第四次，也是同样的结果，这就是幂等性。

同一个请求执行一次和多次对于服务器资源有一样的影响。

根据定义而言：

- HTTP GET方法用于获取资源，不应该有副作用，所以是幂等的。注意，这里说的是，GET方法不会对服务器资源有任何影响，也就没有副作用，并不是指对同一个资源的GET方法会获取一样的数据。（GET方法**没有副作用**，所以是幂等的）
- HTTP DELETE方法用于删除资源，有副作用，但是副作用一致，所以也应该满足幂等性。
- PUT 是幂等的
- POST 不是幂等的

### 容易混淆的PUT和POST

从名字来看，经常会被认为成PUT是用来修改，POST是用来增加的。

而实际上：

| 方法 | 描述                                                                                                                                    |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------- |
| PUT  | 从客户端向服务器传送的数据取代指定的文档的内容。                                                                                        |
| POST | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。 |

从语义化理解而言，PUT用于更新资源，POST用于创建资源，这没什么问题，但实际上，二者最大的区别是幂等性不一致。

POST是不等幂的，PUT是等幂的。

#### why

1. 为什么PUT是符合幂等性的？

其实很简单，PUT是把请求实体放在URL标识下，比如

```javaScript
http.put(url, {
    id: 100,
    name: 'john',
    age: 21
});
```

这个请求会把url位置下，id为100的实体整个替换为请求实例，所以，无论这个PUT请求执行多少次，他对服务器资源的影响都是一样的，所以我们说PUT方法是等幂的。

同时，幂等性属于**语义范畴**，正如编译器只能帮助检查语法错误一样，HTTP规范也没有办法通过消息格式等语法手段来定义它，如果你愿意，甚至可以完全使用GET方法做增删改查。所以，我们在使用HTTP方法的时候，应该尽可能的保证其幂等性，不要随意胡乱使用。

也正是因此，所以当我们使用PUT去对服务器资源进行更新的时候，我们应该传入完整的实体对象，而并不是只需要变更的某几个属性，因为，一旦我们连续请求多次，这期间，此资源可能会被其他资源变更，导致最后的结果不一致，使得幂等性被破坏。

2. 为什么POST是不等幂的？

HTTP POST方法用于创建资源，所对应的URI并非创建的资源本身，而是资源的接收者。比如：

```javaScript
http.post(url, {
    name: 'john',
    age: 21
});
```

这个请求会在URL下创建一个名为john年龄21的实例，如果执行两次，则会创建第二个实例，这两个实例虽然属性一样，但是具有不同的URI，是完全不同的两份资源，所以，这个请求被执行多少次，服务器就会创建多少个资源，每一次的结果都不同，所以，POST方法是不等幂的。

POST实际上也可以用来作为更新资源的方法使用，允许使用POST方法更新部分资源，而不必像PUT一样去传输整个资源来保证幂等性，毕竟，POST是不等幂的。

另外，POST是目前唯一的状态不统一的方法。

*至于PATCH，是HTTP规范还没有完成的另一个方法，意为用于执行部分更新时POST的替代品。但是目前POST已经可以执行不分更新，所以PATCH方法倒显得有些多此一举。一旦PATCH完成标准化，那么PATCH将会是另一个不等幂的HTTP方法。*

#### 幂等性是与浏览器缓存

已经说过，幂等性是指任意N（N >= 1）次执行的影响与一次执行的影响是相同的，所以从另外一个角度而言，幂等操作对于代理和缓存而言是具有“友好性”的，因为幂等操作的额外执行不会对后者产生危害性后果。

当使用相同的URL重复GET请求会返回预期的相同结果时，GET方法才是适用的。 所以浏览器会对GET请求做缓存处理。 

![缓存](https://ws1.sinaimg.cn/large/0064OUUqly1furv10n3gdj30zj01cdft.jpg)

而对于POST请求，则不会被缓存。

基于此情况，如果我们希望某些数据一直从服务端获取，那么可以适当的改为 POST 请求来获取数据。

除此之外，也可以通过像URL后面加时间戳的方法来阻止浏览器读取缓存。

```javaScript
GET url?timestamp=1535621949123
```