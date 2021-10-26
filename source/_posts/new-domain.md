---
title: 网站换域名操作
categories:
  - 信息技术
date: 2020-11-20 20:11:57
tags:
  - DNS
  - Nginx
---

本文描述了网站更换域名的操作

<!-- more -->

2020年8月8日，我买了个新域名，beijing.bb ，我想让我的网站运行在新的域名，同时老域名 beijing2b.com 也可以继续无缝为读者提供服务。但是奇葩小国的效率低得令人发指，我用了两个多月才拿到我的新域名～～
假如不考虑SSL证书的话，DNS服务即可提供重定向。但是我全站开https，只能在Nginx配置上想办法了。
阅读本文需要您了解Nginx基本的配置方法。

### C 准备工作
需要的材料：
- [x] 老域名的DNS权限
- [x] 新域名的DNS权限
- [x] Nginx服务配置权限
- [x] 所有Web内容代码修改权限  
- [ ] 老域名的SSL证书
- [ ] 新域名的SSL证书 
  以上两项虽不强求，但是强烈建议。

### Dm 修改DNS
设我的Web服务器FQDN为：`webserver.beijing2b.com` （此域名不对读者提供服务，直接http发包会被重定向）
读者原来访问的是`www.beijing2b.com`，这是一个CNAME记录，它解析为`webserver.beijing2b.com`
现在加入新的CNAME记录 
`beijing.bb`指向`webserver.beijing2b.com`
`www.beijing.bb`指向`webserver.beijing2b.com`

### Em 修改Nginx配置
找到Nginx配置文件并编辑，比如我的在
```bash
sudo nano  /etc/nginx/sites-enabled/hexo
```
内容参考
```
server {
  #侦听443端口，这个是ssl访问端口
  listen 443;
  #定义使用 访问域名
  server_name beijing.bb;
  #定义服务器的默认网站根目录位置
  root /var/www/hexo/;
  #设定本虚拟主机的访问日志
  access_log logs/nginx.access.log;
  # 这些都是腾讯云推荐的配置，直接拿来用就行了，注意证书的路径即可。
  ssl on;
  ssl_certificate /etc/1_beijing.bb_bundle.crt;
  ssl_certificate_key /etc/2_beijing.bb.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
  ssl_prefer_server_ciphers on;
  #默认请求
  location / {
  root /var/www/hexo/;
      #定义首页索引文件的名称
      index index.html;
  }
  #静态文件，nginx自己处理
  location ~ ^/(images|javascript|js|css|flash|media|static)/ {
      #过期30天，静态文件不怎么更新，过期可以设大一点， 如果频繁更新，则可以设置得小一点。
      expires 30d;
  }
  #禁止访问 .htxxx 文件
  #    location ~ /.ht { deny all;
  #}
}
server {
  # 80端口是http正常访问的接口
  listen 80;
  server_name beijing.bb;
  # 在这里，我做了https全加密处理，在访问http的时候自动跳转到https
  rewrite ^(.*) https://beijing.bb$1 permanent;
}
server {
  #
  listen 443 default_server;
  server_name _;
  # 野生流量拒绝访问
  return 403;
}

server {
    listen       80;
    listen       [::]:80;
    server_name  beijing2b.com www.beijing2b.com;
    #所有旧域名的流量永久重定向
    return       301 https://beijing.bb$request_uri; 
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  beijing2b.com www.beijing2b.com;
    # 这里老域名的SSL证书继续发挥作用。这就避免了证书与域名不匹配的尴尬
    ssl_certificate /etc/1_www.beijing2b.com_bundle.crt;
    ssl_certificate_key /etc/2_www.beijing2b.com.key;

    return       301 https://beijing.bb$request_uri;

}
```
上面的重点是301重定向以及新老域名的SSL证书配置。

### F 修改网站源代码
主要是替换新老域名，各人用的系统不一样，不赘述了。

### G 通知搜索引擎

https://search.google.com/search-console

![new-domain-google](https://cdn.beijing2b.com/new-domain-20201120204523.png)

<center>感谢观看。</center>