---
title: Linux-Nginx安装
date: 2017-11-25 14:48:03
tags: linux
---

### yum安装nginx 
    yum install nginx
  
### nginx 基本命令
    启动 nginx
        service nginx start  或 nginx -s start
        centos7.0+ =>systemctl start nginx
    
    重启 nginx
        service nginx restart  或 nginx -s reload
        centos7.0+ =>systemctl restart nginx
        
    关闭nginx
        service nginx stop 或 nginx -s stop
        centos7.0+ =>systemctl stop nginx
        
### nginx 配置负载均衡

    进入配置文件夹 cd /ect/nginx/
    编辑 vim nginx.conf
    在 http{} 内加入
    upstream myserver{
        server 127.0.0.1:8080 weight=1;
        server quant.tuling.me weight=5;
        server 127.0.0.1:8090 down;
    }
    修改
    location / {
          proxy_pass http://myserver;   #名字和负载均衡相同
    }