---
title: Linux-RibbitMq安装
date: 2018-09-06 16:10:18
tags: linux
---

## 下载RabbitMq：
  
-   Linux: centOS 7.0 版   rabbitMq: 3.7.7
-   rabbitMq服务端最新版网址下载，这里推荐3.7.7 [RibbitMq](http://www.rabbitmq.com/install-generic-unix.html)
-   使用wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.7/rabbitmq-server-generic-unix-3.7.7.tar.xz下载至linux的/app目录下

## 下载Erlang

- 因为rabbitMq是erlang编写的，安装mq前先安装erlang
- 最新版下载网址http://www.erlang.org/downloads
- 使用wget http://erlang.org/download/otp_src_21.0.tar.gz 下载至linux的/app目录下



## Erlang的安装
    1.进入安装目录解压Erlang cd /app tar -xzvf otp_src_21.0.tar.gz
    2.创建erlang安装目录 mkdir /app/erlang
    3.进入解压目录cd /app/otp_src_21.0/
    4.配置安装路径编译代码：./configure --prefix=/app/erlang
    5.如果报No curses library functions found错，安装curses yum -y install ncurses-devel
    6.执行编译结果：make && make install完成后进入/app/erlang查看执行结果    
    7.配置Erlang环境变量,vi /etc/profile文件，增加下面的环境变量:
      export PATH=$PATH:/app/erlang/bin
    8.使得文件生效 source  /etc/profile 至此erlang安装完成
    9.验证erlang是否安装成功：erl
      退出erl:halt();
    
## RabbitMq安装
    1.因为下载的是tar.xz文件所以先用xz -d rabbitmq-server-generic-unix-3.7.7.tar.xz解压成rabbitmq-server-generic-unix-3.7.7.tar
    然后再用tar -xvf rabbitmq-server-generic-unix-3.7.7.tar
    2.添加环境变量：export PATH=$PATH:/app/rabbitmq_server-3.7.7/sbin
    3.环境变量生效：source  /etc/profile
    4.进入sbin 启动服务：./rabbitmq-server -detached 后台启动   |start前台启动
    5.如果启动报错："init terminating in do_boot",{error,{missing_dependencies,[crypto,ssl]
    6.根据异常提示，原先以为是缺少了OpenSSL，但是检查后发现OpenSSL已经成功安装
      $ openssl version
      OpenSSL 1.0.2g  1 Mar 2016
    7.为什么会缺失crypto和ssl呢，百度，发现缺失了erlang-ssl但是yum找不到！！！在配置openssl后在Makefile中的CFLAG配置项后边添加 -fPIC参数。
    -fPIC（作用于编译阶段，告诉编译器产生与位置无关代码(Position-Independent Code)，则产生的代码中，没有绝对地址，全部使用相对地址，故而代码可以被加载器加载到内存的任意位置，都可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。）
    8.找到原因后重新安装openssl wget http://www.openssl.org/source/openssl-1.0.1i.tar.gz  
      tar -zxf openssl-1.0.1i.tar.gz   
      cd openssl-1.0.1i  
      ./config --prefix=/app/ssl    
      sed -i "s|CFLAG= |CFLAG= -fPIC |" Makefile  #一定不能少了这步
      make && make install
      9.再配置安装erlang时指定刚才安装的open-ssl地址  ./configure --prefix=/app/erlang --prefix=/app/erlang  
      10.rabbitmq-server -detached  总算启动成功了
      
## 配置网页插件

- 首先创建目录，否则可能报错：mkdir /etc/rabbitmq 

- 进入sbin 启用插件：./rabbitmq-plugins enable rabbitmq_management

- 启动mq：./rabbitmq-server -detached 后台启动| start 前台启动

- 配置linux 端口： 15672 网页管理，  5672 AMQP端口

- 然后访问http://127.0.01:15672

- rabbitmq默认会创建guest账号，只能用于localhost登录页面管理员

> 进入sbin
- 查看服务状态：rabbitmqctl status

- 关闭服务：rabbitmqctl stop

- 查看mq用户：rabbitmqctl list_users  

- 查看用户权限：rabbitmqctl list_user_permissions guest

- 新增用户： rabbitmqctl add_user admin 123456

- 赋予管理员权限：(需要服务启动)

- rabbitmqctl set_user_tags admin administrator 

- rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"

- 至此安装完成 

 