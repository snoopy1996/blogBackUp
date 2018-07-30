---
title: nginx相关
date: 2018-07-28 21:43:17
tags: [nginx, 技术]
---

# 基础

### 服务地址

```
/etc/init.d/nginx
```

<!--more-->

### 配置地址

```
/etc/nginx/
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqi2omd6t5j30d6080aah.jpg)

### web 默认目录

```
/var/www/html
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqi2plev9uj30fp02p3yj.jpg)

### 日志目录

```
/var/log/nginx
```

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqi2qtk98ij30ek03pt8s.jpg)

## 配置方式

### 简单静态配置文件

![](https://ws1.sinaimg.cn/large/0064OUUqly1fqi2s70neuj30kc0ct0uo.jpg)

### 配置 demo

- 通过http://localhost:8080/ 访问一个存放于磁盘位置：/var/www/html/ 下的网站
- 通过http://localhost:80/ 访问一个存放于磁盘位置：/var/www/myweb/ 下的网站

```
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;
    # SSL configuration
    #
    # listen 443 ssl default_server;
    # listen [::]:443 ssl default_server;


    root /var/www/html;

    # 首页配置
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
server {
    listen 80;
    listen [::]:80;
    server_name myweb;

    root /var/www/myweb;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 命令

- 查看**nginx**进进程

```
Operation not permitted
```

`绿框是主进程`
![](https://ws1.sinaimg.cn/large/0064OUUqly1fqi2liujccj30qr01p0sr.jpg)

- 重载配置

```
sudo nginx -s reload
```

- 重启服务

```
sudo service nginx restart
```
