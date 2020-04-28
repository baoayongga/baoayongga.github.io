---
title: Typecho在Apache环境下开启伪静态
date: 2018-12-27 14:55:52
tags:
    - Linux
    - Apache
    - Typecho
categories: [分享经验]
---

```
本文环境:
CentOS7.3
Apache 2.4.6
PHP 5.4.16
网站存放位置: /var/www/html
```

<!-- more -->

## 1、修改Apache的rewrite模块
`vi /etc/httpd/conf/httpd.conf`


找到httpd.conf中是否包含`LoadModule rewrite_module modules/mod_rewrite.so`如果没有就自己加一个

增加配置如下:
```
<Directory "/var/www/html"> 
    Order allow,deny
    Allow from all
    AllowOverride All
</Directory>
```
## 2、修改.htaccess文件

路径：/var/www/html/.htaccess，如果没有就新建一个将以下内容添加进去。

```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ index.php [E=PATH_INFO:$1] 
</IfModule>
```

**重启apache**

## 3、开启网站地址重写功能

![地址重写功能.png][1]

如果提示“重写功能检测失败, 请检查你的服务器设置” 勾上继续就行


  [1]: https://www.snasec.com/usr/uploads/2019/02/270287826.png