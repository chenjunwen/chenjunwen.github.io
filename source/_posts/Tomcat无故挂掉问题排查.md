---
title: Tomcat无故挂掉问题排查
date: 2017-11-25 16:27:03
tags: tomcat
---

## 查找正在运行tomcat进程
    ps aux | grep tomcat
 
## 查看某个进程中各个线程的使用cpu的排行
    top -Hp 3306
    
## 假设现在比较高的线程为3307，找到正在执行的方法（通过包名搜索）
    jstack 3306|grep movitech
    
## 查询类名，对象数量，对象占用大小
    jmap -histo:live 3306|more

## 通过 jstat -gcutil 来学习JVM 内存分配策略与 GC 发生时机[URL](https://www.cnblogs.com/orientsun/archive/2012/07/25/2608545.html)
    jstat -gcutil 3306 

## %x即按十六进制输出，英文字母小写，右对齐
    printf %x 0x1ff4    

## 实例分析: [URL](https://www.cnblogs.com/wuchanming/p/7766994.html)
## JVM中的新生代和老年代（Eden空间、两个Survior空间）[URL](https://blog.csdn.net/jisuanjiguoba/article/details/80156781)


## jps、jinfo、jstat、jstack、jmap、jconsole等命令简介[URL](https://my.oschina.net/guoenzhou/blog/389687)
