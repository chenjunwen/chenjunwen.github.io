---
title: Linux-Docker安装
date: 2019-12-18 15:09:50
tags: linux
---

## 安装环境
    CentOS 6.5 (64-bit) 或更高的版本
    
## 使用 yum 安装（CentOS 7下）
    Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。
    通过 uname -r 命令查看你当前的内核版本，如果大于3.1则可以使用域名安装

## 使用yum安装docker

    yum install docker -y
        
## docker基本命令

    service docker start/systemctl start mysqld #启动
    service docker stop #停止
    service docker restart #重启
    service docker status #状态
## 设置开机启动
    systemctl enable docker
    systemctl daemon-reload
    
## 
    

