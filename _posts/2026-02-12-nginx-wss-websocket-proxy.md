---
layout: post
title:  "Nginx 配置 WSS 反向代理：WebSocket 安全连接完整指南"
date:   2026-02-12 03:32:43 +0800
categories: nginx
tags: nginx websocket wss 反向代理 ssl 宝塔
author: 来财
---

本文详细介绍了如何使用 Nginx 配置 WSS（WebSocket Secure）反向代理，将 HTTPS 请求转发到后端的 WebSocket 服务。包含基础配置、宝塔面板实操步骤以及完整的生产环境配置示例，帮助你快速搭建安全可靠的 WebSocket 代理服务。

* content
{:toc}





这样可以把wss请求转发到后端的ws请求上


```
map $http_upgrade $connection_upgrade {  
    default upgrade;  
    '' close;  
}  
upstream websocket {  
    server 128.190.82.105:8888;  
}  
server {  
    listen 8888;  
    server_name proxy.hello.com;
    ssl on;
    ssl_certificate /etc/nginx/ssl/hello.com_bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/hello.com.key;
    ssl_session_timeout 20m;
    ssl_verify_client off;
    location / {  
        proxy_pass http://websocket;  
        proxy_http_version 1.1;  
        proxy_set_header Upgrade $http_upgrade;  
        proxy_set_header Connection "Upgrade";  
    }  
}
```


宝塔亲测可用
1. 建立一个纯静态网站
2. 给网站配置ssl，必须先配置这个，后面再开反向代理
3. 打开反向代理到websocket监听服务上
4. 重启NGINX


```
#PROXY-START/
    location ~* \.(php|jsp|cgi|asp|aspx)$
    {
    	proxy_pass http://8.129.108.137:9504;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header REMOTE-HOST $remote_addr;
    }
    location /
    {
        proxy_pass http://8.129.108.137:9504;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header REMOTE-HOST $remote_addr;
        
        add_header X-Cache $upstream_cache_status;
        
        #Set Nginx Cache
        
        	add_header Cache-Control no-cache;
        expires 12h;
    }
    #socket.io默认的地址
    location /socket.io {
      proxy_pass http://8.129.108.137:9504;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }
#PROXY-END/
```


### 完整示例

```
map $http_upgrade $connection_upgrade {  
    default upgrade;  
    '' close;  
} 
upstream wss {
  # 这里的localhost是映射本地服务器，也可以是外网ip，2345是我ws开启的端口。
  server localhost:8273;
}

server
{
    listen 80;
		listen 443 ssl http2;
    server_name cheguan.bonza.cn tjfeng.bonza.cn;
    index index.php index.html index.htm default.php default.htm default.html;
    root /www/wwwroot/cheguan.bonza.cn/public;

    #SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则
    #error_page 404/404.html;
    ssl_certificate    /www/server/panel/vhost/cert/cheguan.bonza.cn/fullchain.pem;
    ssl_certificate_key    /www/server/panel/vhost/cert/cheguan.bonza.cn/privkey.pem;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    add_header Strict-Transport-Security "max-age=31536000";
    error_page 497  https://$host$request_uri;


    #SSL-END

    #ERROR-PAGE-START  错误页配置，可以注释、删除或修改
    #error_page 404 /404.html;
    #error_page 502 /502.html;
    #ERROR-PAGE-END

    #PHP-INFO-START  PHP引用配置，可以注释或修改
    include enable-php-74.conf;
    #PHP-INFO-END

    #REWRITE-START URL重写规则引用,修改后将导致面板设置的伪静态规则失效
    include /www/server/panel/vhost/rewrite/cheguan.bonza.cn.conf;
    #REWRITE-END

    #禁止访问的文件或目录
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.env|\.svn|\.project|LICENSE|README.md)
    {
        return 404;
    }

    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }

    #禁止在证书验证目录放入敏感文件
    if ( $uri ~ "^/\.well-known/.*\.(php|jsp|py|js|css|lua|ts|go|zip|tar\.gz|rar|7z|sql|bak)$" ) {
        return 403;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires      30d;
        error_log /dev/null;
        access_log /dev/null;
    }

    location ~ .*\.(js|css)?$
    {
        expires      12h;
        error_log /dev/null;
        access_log /dev/null;
    }
    location /wss  {   
       proxy_pass http://wss;        #通过配置端口指向部署websocker的项目
       proxy_http_version 1.1;    
       proxy_set_header Upgrade $http_upgrade;    
       proxy_set_header Connection "Upgrade";    
       proxy_set_header X-real-ip $remote_addr;
       proxy_set_header X-Forwarded-For $remote_addr;
     }
    access_log  /www/wwwlogs/cheguan.bonza.cn.log;
    error_log  /www/wwwlogs/cheguan.bonza.cn.error.log;
}
```
