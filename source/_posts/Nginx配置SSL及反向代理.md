---
title: Nginx配置SSL及反向代理
date: 2019-01-16 19:52:45
tags: [nginx, https, ssl, 反向代理]
---

最近搞毕设，前端准备使用 PWA，而 PWA 的一项要求就是网站应该是`https`而非`http`的，所以就查了一些资料，做一下简单的配置记录。

<!--more-->

### 所用环境

1. Ubuntu1604
2. Nginx

### 所需资料

已备案域名一枚~

### SSL 证书获取

> SSL 证书是数字证书的一种，类似于驾驶证、护照和营业执照的电子副本。因为配置在服务器上，也称为 SSL 服务器证书。

> SSL 证书就是遵守 SSL 协议，由受信任的数字证书颁发机构 CA，在验证服务器身份后颁发，具有服务器身份验证和数据传输加密功能。

国内来说，一般个人购买服务器会和域名同时在一个服务商处购买，目前国内的云服务器也比较多，阿里云，腾讯云，百度云等。

笔者这里使用的是腾讯云处购买备案的域名。

#### 为域名注册 SSL 证书

[腾讯云 SSL 证书注册地址](https://cloud.tencent.com/product/ssl?from=qcloudHpHeaderSsl)

对于个人用户，可以申请免费的个人 SSL 证书，这个证书有效期是一年，单域名使用。（腾讯云每个域名最多申请 20 个免费 SSL 证书）

![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8nck37zuj31150fmn0m.jpg)
![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8nd746ggj30zo0kz0uo.jpg)
跟随提示，一步步向下填写即可。
![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8ne8b29hj30hl0ijmyf.jpg)

申请提交后，就是待验证状态，这个一般很快，几分钟就 OK
![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8nfhcknfj317p02c748.jpg)

然后点击下载，将已经颁发的证书下载到本地
![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8ngpf7gej317502jglk.jpg)

解压缩后，里面已经内置了四种服务器的需要的证书文件和秘钥，使用的时候，直接把对应文件夹的内容上传到服务器对应的位置即可。
![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8ngzwefqj30px04rglu.jpg)

### Nginx 配置

#### 上传证书

将解压后的 Nginx 文件夹里的内容上传到 `/etc/nginx/cert` 文件夹下。`cert` 是个人新建的文件夹，也可以改成其他的名字。
![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8nmc4vsmj30je043t8v.jpg)

#### 配置内容

在`sites-enabled`文件夹下新建一个配置文件，为管理方便，命名为要注册的域名即可，以 `qnote.supersy.xyz` 为例：

![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8noeq3euj30ho03ct8p.jpg)

```
server {
    listen 443; # 监听443端口(https)
    server_name qnote.supersy.xyz;  # 域名
    ssl on;
    root /var/www/qnote;    # 如果是静态资源，这里就是静态资源的所在文件夹
    index index.html index.htm; # 首页文件
    ssl_certificate  cert/certname.crt; # crt文件名
    ssl_certificate_key cert/keyname.key;   # key文件名
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    # 反向代理配置
    location / {
       proxy_pass http://localhost:8000;    # 需要代理的ip和端口
        proxy_set_header Host      $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_redirect http:// $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
server {
    listen 80;
    server_name qnote.supersy.xyz;
    rewrite ^(.*)$ https://$host$1 permanent;   # 将http重写为https
}
```

#### 检测与重启

```bash
# 此命令可以测试配置文件的有效性，出现 successful 即说明配置成功
nginx -t
# 重载配置文件
nginx -s reload
# 重启nginx
service nginx restart
```

#### 浏览器打开域名地址

![](https://ws1.sinaimg.cn/large/0064OUUqly1fz8nx60pupj30ey01b3yc.jpg)
