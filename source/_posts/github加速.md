---
title: github加速
date: 2023-07-06 13:50:45
tags: 工具
---


## 1.1 UsbEAm Hosts Editor
由羽翼城大佬开发的知名项目UsbEAm Hosts Editor本来是用于改善 Steam、暴雪、育碧、Microsoft Store 等游戏平台的访问与下载速度，但顺便也有支持 Github

软件可以直接到大佬博客上进行下载：https://www.dogfight360.com/blog/475/

打开软件，点击软件左下角，选择准备修改 hosts 的网站

## 1.2 Github520＋SwitchHosts
Github520 的作者也长期在维护 Github 的优质IP地址，并且是动态更新的：
https://github.com/521xueweihan/GitHub520

SwitchHosts 是一个管理 hosts 文件的应用，基于 Electron 、React、Jotai 、Chakra UI、CodeMirror 等技术开发
https://github.com/oldj/SwitchHosts/blob/master/README.zh_hans.md


## 1.3 steamcommunity 302（WIN）
羽翼城大佬后来又开发的 steamcommunity 302 就是通过反代来加速访问 Github 等网站

软件可以到这里下载：https://www.dogfight360.com/blog/686/

在不遇到问题的情况下可谓想当无脑，一键化使用


## 1.4 FastGithub（WIN／Mac／Linux）
steamcommunity 302 很不错但缺点是只有Windows端

FastGithub 则是另一款同样基于反代来加速 Gtihub 访问的工具，支持WIN／Mac／Linux三端，还能在docker上一键部署：https://github.com/dotnetcore/FastGithub

Windows端的话，下载后双击即可运行，软件没有程序界面，直接就是跑的命令行，所以开启之后不要关闭命令行窗口！

## 1.5 dev-sidecar（WIN／Mac／Ubuntu／Linux）
最后再介绍一款反代工具吧，dev-sidecar 这个项目命名取自service-mesh的service-sidecar，意为为开发者打辅助的边车工具，主要就是用于解决 Github 访问的问题：https://github.com/docmirror/dev-sidecar