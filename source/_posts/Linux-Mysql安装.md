---
title: Linux-Mysql安装
date: 2017-11-06 14:35:29
tags: linux
---
## 配置yum源
在MySQL官网中下载YUM源rpm安装包：

    rpm -ivh https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
    这里下载的是mysql.8
## 使用yum安装mysql

    yum install mysql-server -y
        
## mysql基本命令

    service mysqld start/systemctl start mysqld #启动
    service mysqld stop #停止
    service mysqld restart #重启
    service mysqld status #状态
## 设置开机启动

    systemctl enable mysqld
    systemctl daemon-reload
## 修改root默认密码
mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：
    
    grep 'temporary password' /var/log/mysqld.log

![image](http://www.centoscn.com/uploads/allimg/160626/1-160626010T0Z8.jpg)

    mysql -uroot -p
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!'; 
    或者
    mysql> set password for 'root'@'localhost'=password('MyNewPass4!');
    
    
    密码必须包含大小写数组（否则需要修改配置）
    
## 配置默认编码为utf8
    
>修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：

    [mysqld]
    character_set_server=utf8
    init_connect='SET NAMES utf8'
    重新启动mysql服务，查看数据库默认编码如下所示：
![image](http://www.centoscn.com/uploads/allimg/160626/1-160626010915C6.jpg)


## 配置外部网络访问

> 登录 默认root用户登录 命令全名为 mysql -u root -p你的密码

    # 允许所有ip以root用户登录mysql
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '你的密码';
    # 允许本地ip已root用户登录mysql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';

> 如果有防火墙的话得打开3306端口
    
    # 允许3306端口外网访问
    firewall-cmd --zone=public --add-port=3306/tcp --permanent
    # 重新加载防火墙配置
    firewall-cmd --reload


## 默认配置文件路径
> 配置文件：/etc/my.cnf  日志文件：/var/log//var/log/mysqld.log  服务启动脚本：/usr/lib/systemd/system/mysqld.service  socket文件：/var/run/mysqld/mysqld.pid
## [辅助文档](http://www.centoscn.com/mysql/2016/0626/7537.html)    