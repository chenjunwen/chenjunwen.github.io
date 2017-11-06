---
title: Linux-Jdk安装
date: 2017-11-06 14:37:25
tags: linux
---

## 1.下载
 
    wget http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
    linux下载较慢建议使用浏览器下载后上传
    
## 2.解压
    
    tar -zxvf *.tar.gz

## 3.配置环境变量
    打开/etc/profile（vim /etc/profile）
    在最后面添加如下内容：
    JAVA_HOME=/usr/java/jdk1.7.0_40
    CLASSPATH=.:$JAVA_HOME/lib.tools.jar
    PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME CLASSPATH PATH
    
## 4.使配置生效
    source /etc/profile

## 5.验证是否安装成功
    java -version