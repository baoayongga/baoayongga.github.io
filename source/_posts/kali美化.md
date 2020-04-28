---
title: Kali美化
date: 2019-03-18 22:20:52
tags:
    - Linux
    - 美化
categories: [分享经验]
---

**kali自带的界面太丑了，终于忍不住下手了**

<!-- more -->


## 更换主题

安装`gnome-tweak-tool`

```
apt-get install gnome-tweak-tool
```


kali主题下载地址

> http://gnome-look.org

将下载好的theme放入`/usr/share/themes`


博主的主题是

>https://github.com/vinceliuice/Qogir-theme

使用方式：
```
git clone https://github.com/vinceliuice/Qogir-theme.git

sudo apt-get install gtk2-engines-murrine gtk2-engines-pixbuf

cd Qogir-thme

./install
```

## 更换图标

将下载好的图标文件放入

`/usr/share/icons`

![AnFe29.png](https://s2.ax1x.com/2019/03/18/AnFe29.png)


终端输入`gnome-tweaks`


![AniRAO.png](https://s2.ax1x.com/2019/03/18/AniRAO.png)


搞定！



