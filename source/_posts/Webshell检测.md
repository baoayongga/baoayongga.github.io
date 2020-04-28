---
title: Webshell检测
date: 2020-03-29 21:00:52
tags:
   - Webshell
   - 一句话木马
   - 中国菜刀
categories: [学习笔记]
---

# WebShell简介

Web指的是在web服务器上，而shell是用脚本语言编写的脚本程序，webshell就是就是Web的一个管理工具，可以对web服务器进行操作的权限，也叫webadmin。

<!-- more -->

webshell一般是被网站管理员用于网站管理、服务器管理等等一些用途，但是由于webshell的功能比较强大，可以上传下载文件，查看数据库，甚至可以调用一些服务器上系统的相关命令（比如创建用户，修改删除文件之类的），通常被黑客利用，黑客通过一些上传方式，将自己编写的webshell上传到web服务器的页面的目录下，然后通过页面访问的形式进行入侵，或者通过插入一句话连接本地的一些相关工具直接对服务器进行入侵操作。

## Webshelld分类
小马：体积小，代码量少，常见的一句话木马`<?php @eval($_POST['attack']) ?>`，可以通过中国菜刀等工具去连接操作。

大马：体积大，功能齐全；通常会使用系统关键函数

![大马](https://s1.ax1x.com/2020/03/29/GZ9VpV.png)

常见的像asp、jsp、php等编程语言编写的shell脚本

![Webshell特征](https://s1.ax1x.com/2020/03/29/GZP6eg.png)

## WebShell管理工具

- 中国蚁剑AntSword
- 中国菜刀 (caidao)
-  冰蝎 webshell
-  cknife (c刀)
-  xise
-  QuasiBot
-  WeBaCoo
-  Weevely
-  Fastener
-  Altman
-  Webshell-Sniper

## Webshell检测方法

### 静态检测

静态检测指对脚本文件中所使用的特征码，特征值、关键词、高危函数、文件修改的时间、文件权限、文件的所有者以及和其它文件的关联性等多个维度的特征进行检测，

- 缺点
  - 只能查找已知的webshell
  - 误报率漏报率较高
  - 容易被绕过

- 优点：
  - 快速方便，对已知的webshell查找准确率高
  - 部署方便
  - 一般情况下需要配合人工

### 动态检测

动态特征检测通过Webshell运行时使用的系统命令或者网络流量及状态的异常来判断动作的威胁程度，Webshell通常会被加密从而避免静态特征的检测，当Webshell运行时就必须向系统发送系统命令来达到控制系统或者操作数据库的目的，通过检测系统调用来监测甚至拦截系统命令被执行，从行为模式上深度检测脚本文件的安全性。

- 优点：
  - 可用于网站集群，对新型变种脚本有一定的检测能力。

- 缺点：
  - 针对特定用途的后门较难检测,实施难度较大。

### 日志分析

使用Webshell一般不会在系统日志中留下记录，但是会在网站的web日志中留下Webshell页面的访问数据和数据提交记录。日志分析检测技术通过大量的日志文件建立请求模型从而检测出异常文件，称之为：HTTP异常请求模型检测。例如：一个平时是GET的请求突然有了POST请求并且返回代码为200、某个页面的访问者IP、访问时间具有规律性等。

- 优点：
  - 采用了一定数据分析的方式，网站的访问量达到一定量级时这种检测方法的结果具有较大参考价值。

- 缺点：
  - 存在一定误报，对于大量的访问日志，检测工具的处理能力和效率会比较低。

### 语法检测

根据php语言扫描编译的实现方式，进行剥离代码、注释，分析变量、函数、字符串、语言结构的分析方式，来实现关键危险函数的捕捉方式。

## 变形、窃密型webshell检测

### 基于数据库操作审计的检测方式

针对窃密型Webshell必须具有操作数据库的能力，可以引申出一种新的检测方法，通过分析正常WEB脚本文件和窃密型Webshell对数据库操作的差异进行分析是本检测方法所重点研究的方向。正常情况下WEB站点进行数据操作的过程应该是重复性且较为复杂的查询过程，这种查询通常精确度非常高，查询过程不会出现类似于“select * from”这种查询语句。正常的WEB脚本在进行数据库操作的过程中也不会出现跨越数据库查询的情况，一旦出现这种现象基本可以判断为非正常的WEB脚本操作过程。

### **建立机器学习日志分析系统**

由于数据库操作记录日志量非常大，使用人工的方法难以进行精确筛选和审计。所以需要建立一套机器自学习的日志审计系统。该日志审计系统主要基于查询模型白名单学习与数学统计模型这两方面进行设计。



## Webshell检测工具

### D盾

支持系统:win2003/win2008/win2012/win2016
PHP支持:FastCGI/ISAPI (版本:5.x至7.x)

[点击下载](http://www.d99net.net/down/d_safe_2.1.5.4.zip) | [安装与使用说明](http://www.d99net.net/News.asp?id=106)

![D盾](https://s1.ax1x.com/2020/03/29/GZVH0K.png)

### WEBDIR+：

需要用户自行上传可疑文件。

支持的文件类型 php, phtml, inc, php3, php4, php5, war, jsp, jspx, asp, aspx, cer, cdx, asa, ashx, asmx, cfm

支持的压缩包 rar, zip, tar, xz, tbz, tgz, tbz2, bz2, gz

![](https://s1.ax1x.com/2020/03/29/GZZKBV.png)

### WebShell Detector

可以识别和发现php/cgi(perl)/asp/aspx的shell

![WebShell Detector](https://s1.ax1x.com/2020/03/29/GZZx5F.png)

### WEBSHELL.PUB

GUI/在线WebShell扫描检测查杀

![河马](https://s1.ax1x.com/2020/03/29/GZeyIU.png)

* 参考来源

  [webshell检测方法归纳](https://www.cnblogs.com/he1m4n6a/p/9245155.html "webshell检测方法归纳")