---
title: NetEase-MusicBox
date: 2018-12-26 12:32:23
tags:
    - Linux
    - 终端
    - 音乐
categories: [工具分享]
---


## 基于Python编写的 高品质网易云音乐命令行版本

<!-- more -->

![示范.gif][1]

## 安装

### Ubuntu/Debian
```
$ (sudo) pip install NetEase-MusicBox

$ (sudo) apt-get install mpg123
```
## 可选依赖
`aria2` 用于缓存歌曲
`libnotify-bin` 用于支持消息提示（Linux平台）
`pyqt python-dbus dbus qt` 用于支持桌面歌词 

## 使用
```
$ musicbox
```

*参考来源：[musicbox][2]*


  [1]: https://camo.githubusercontent.com/a2cdeab17a7ea0218ec84b25c1026d8bf5483f29/68747470733a2f2f7166696c652e616f626565662e636e2f33616262613362386133393934656533643563642e676966
  [2]: https://github.com/darknessomi/musicbox