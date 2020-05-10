---
title: Linux/Windows入侵排查
date: 2020-5-10 19:10:23
tags:
    - Linux
    - Windows
    - 应急响应
    - 入侵排查
categories: [学习笔记]
---

## 一、Linux 排查方法
### 1.1 系统完整性
 `rpm -Va`
![rpm-Va](https://na1r.oss-cn-beijing.aliyuncs.com/20200505215014.png)
**说明：**

<!-- more -->

S 表示文件长度发生了变化
M 表示文件的访问权限或文件类型发生了变化
5 表示MD5校验和发生了变化
D 表示设备节点的属性发生了变化
L 表示文件的符号链接发生了变化
U 表示文件/子目录/ 设备节点的owner 发生了变化
G 表示文件/子目录/ 设备节点的group 发生了变化
T 表示文件最后一次的修改时间是发生了变化

### 1.2 进程
`top`
显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。
![top](https://na1r.oss-cn-beijing.aliyuncs.com/20200505215227.png)

`ps -aux`
系统运行的全部进程，可以用来寻找可以的进程
![ps -aux](https://na1r.oss-cn-beijing.aliyuncs.com/20200505215548.png)
[ps -aux命令详解](https://blog.csdn.net/x_i_y_u_e/article/details/48463623)

`lsof`
可以查进程、端口、网络连接等等，lsof是最多开关的Linux/Unix命令之一。

lsof语法格式是： 
```
lsof ［options］ filename
```
![lsof](https://na1r.oss-cn-beijing.aliyuncs.com/20200505220245.png)

[Linux 命令神器：lsof](https://www.jianshu.com/p/a3aa6b01b2e1)
### 1.3 网络连接
`netstat -pantu`
查看当前所有网络连接，可以用来寻找异常的连接
![netstat -pantu](leanote://file/getImage?fileId=5eb17293a0de1675c4000000)
### 1.4 开机启动项
```
/etc/init.d/*
/etc/rc*.d/
/etc/rc.local
```
### 1.5 WebShell查杀
```
河马查杀、D盾、自写脚本等
```
### 1.6 计划任务
```
cat /var/log/cron |grep "bash"
cat /var/log/cron |grep "wget"
cat /var/log/cron |grep "cat"
cat /var/log/cron |grep "id"
cat /var/log/cron |grep -E -o "(([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3}))"

cat /etc/crontab
/var/spool/cron/*
/etc/cron.d/
/etc/cron.daily
/etc/cron.hourly
/etc/cron.weekly
/etc/cron.monthly
```
### 1.7 登录情况
历史登录成功IP及账号，当前登录成功账号IP
```
cat /var/log/secure
last
```
### 1.8 SSH key
查看是否存在免密登录
```
cat .ssh/aythorized_key
```
### 1.9 用户及hosts
```
/etc/passwd
/etc/hosts/
```
### 1.10 动态链接库
```
echo $LD_PRELOAD

Linux预加载库
/etc/ld.so.preload
/usr/local/lib
```
可以通过ldd 命令查看 有没有一些恶意的动态链接库
![动态链接库](https://na1r.oss-cn-beijing.aliyuncs.com/20200505223228.png)
### 1.11 日志分析
Linux /var/log下的各种日志文件
```
/var/log/secure：记录登录系统存取数据的文件

/var/log/btmp：记录登录者的信息记录，被编码过，所以必须以last解析

/var/log/message:几乎所有的开机系统发生的错误都会在此记录

/var/log/cron：用来记录crontab这个服务的内容

/var/run/utmp 记录着现在登录的用户

/var/log/lastlog 记录每个用户最后的登录信息

/var/log/btmp 记录错误的登录尝试

/var/log/syslog 事件记录监控程序日志
```
**常用命令：**
1. 查看有多少IP在爆破`root`账户：
```
grep "Failed password for root" /var/log/secure |awk '{print $11}' |sort |uniq -c | sort -nr | more
```
-
2. 查看登录成功的IP
```
grep "Accepted " /var/log/secure |awk '{print $11}' |sort |uniq -c |sort -nr |more
```

## 二、Windows排查方法
### 2.1 查看系统状态
CPU、内存、网络状态等
![任务管理器](https://na1r.oss-cn-beijing.aliyuncs.com/20200505224717.png)

### 2.2 文件分析

#### a. 临时目录
黑客往往可能将病毒放在临时目录（tmp/temp），或者将病毒相关文件释放到临时目录，因此需要检查临时目录是否存在异常文件。

```
假设系统盘在C盘，则通常情况下的临时目录如下：
C:\Users\[用户名]\AppData\Local\Temp
C:\Users\[用户名]\Local Settings\Temp
C:\Documents and Settings\[用户名]\Local Settings\Temp
C:\Users\[用户名]\桌面
C:\Documents and Settings\[用户名]\桌面
C:\Users\[用户名]\Local Settings\Temporary Internet Files
C:\Documents and Settings\[用户名]\Local Settings\Temporary Internet Files
```

#### b. 浏览器相关文件
需要检查下浏览器的历史访问记录、文件下载记录、cookie信息，对应相关文件目录如下：
```
C:\Users\[用户名]\Cookies
C:\Documents and Settings\[用户名]\Cookies
C:\Users\[用户名]\Local Settings\History
C:\Documents and Settings\[用户名]\Local Settings\History
C:\Users\[用户名]\Local Settings\Temporary Internet Files
C:\Documents and Settings\[用户名]\Local Settings\Temporary Internet Files
```
#### c. 最近文件
检查最近打开了哪些文件，可疑文件有可能就在最近打开的文件夹中，打开以下这些目录即可看到：
```
C:\Users\[用户名]\Recent
C:\Documents and Settings\[用户名]\Recent

单击【开始】>【运行】，输入%UserProfile%\Recent，分析最近打开分析可疑文件
```
#### d. 文件修改时间
可以根据文件夹内文件列表时间进行排序，查找可疑文件。一般情况下，修改时间越近的文件越可疑。当然，黑客也有可能修改”修改日期“。
![修改日期](https://na1r.oss-cn-beijing.aliyuncs.com/20200510173931.png)
#### e.System32目录与hosts文件
System32也是常见的病毒释放目录，因此也要检查下该目录。hosts文件是系统配置文件，用于本地DNS查询的域名设置，可以强制将某个域名对应到某个IP上，因此需要检查hosts文件有没有被黑客恶意篡改。
```
C:\Windows\System32
C:\Windows\System32\drivers\hosts
```
### 2.3 网络连接
1. 使用命令`netstat -ano`查看目前的网络连接，定位可疑的ESTABLISHED。
推荐工具：D盾
2. 根据netstat 定位出的pid，再通过`tasklist`命令进行进程定位 `tasklist | findstr PID`
![netstat](https://na1r.oss-cn-beijing.aliyuncs.com/20200510175012.png)
![tasklist](https://na1r.oss-cn-beijing.aliyuncs.com/20200510175137.png)

### 2.4 流量分析
流量分析可以使用Wireshark，主要分析下当前主机访问了哪些域名、URL、服务，或者有哪些外网IP在访问本地主机的哪些端口、服务和目录，又使用了何种协议等等。
例如，使用Wireshark观察到，主机访问了sjb555.3322.org这种动态域名，即可粗略猜测这是一个C&C服务器
![恶意链接](https://na1r.oss-cn-beijing.aliyuncs.com/20200510175553.png)
知道域名地址后可以使用威胁情报平台进行查询
![微步在线](https://na1r.oss-cn-beijing.aliyuncs.com/20200510175710.png)
### 2.5 漏洞及补丁信息
使用命令`systeminfo`，查看系统版本信息以及补丁信息，确认当前系统是否存在漏洞、是否已经打了相应的补丁。
![systeminfo](https://na1r.oss-cn-beijing.aliyuncs.com/20200510180211.png)
### 2.6 可疑进程分析
#### a. 异常的进程名
#### b. 进程信息分析
可以使用PCHunter、火绒剑等软件。
### 2.7 检查启动项、计划任务、服务
黑客为了保持病毒能够开机启动、登录启动或者定时启动，通常会有相应的启动项，因此有必要找出异常启动项，并删除。启动项的排查
工具推荐：Autoruns（官网www.sysinternals.com）
![Autoruns](https://na1r.oss-cn-beijing.aliyuncs.com/20200510183426.png)
火绒剑等。
### 2.7 日志排查
#### a. 打开事件管理器
```
【开始】➜【管理工具】➜【事件查看器】
【开始】➜【运行】➜【eventvwr】
```
![事件查看器](https://na1r.oss-cn-beijing.aliyuncs.com/20200510191232.png)
#### b. 主要分析安全日志，可以借助自带的筛选功能
![筛选日志](leanote://file/getImage?fileId=5eb7e1e3fa27952abc000002)

### tips
#### 删除guest用户
a.  "win + r" 输入regedit打开注册表

b.  找到HKEY_LOCAL_MACHINE\SAM\SAM，右键 `权限` 设置管理员->完全控制权限
![1](https://na1r.oss-cn-beijing.aliyuncs.com/20200510183919.png)
c. 关闭注册表，
d. 打开HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names\Guest
![2](https://na1r.oss-cn-beijing.aliyuncs.com/20200510184449.png)

d.从上图可知type值为0x1f5，这里我们删除
“HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\000001F5”和“HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names\Guest”两个键就可以了。
![3](https://na1r.oss-cn-beijing.aliyuncs.com/20200510184639.png)
## 常用工具及命令
### 1.病毒分析 ：
> PCHunter：http://www.xuetr.com
> 火绒剑：https://www.huorong.cn
> Process Explorer：https://docs.microsoft.com/zh-cn/sysinternals/downloads/process-explorer
> processhacker：https://processhacker.sourceforge.io/downloads.php
> autoruns：https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns
> OTL：https://www.bleepingcomputer.com/download/otl/
### 2. 病毒查杀：
> 卡巴斯基：http://devbuilds.kaspersky-labs.com/devbuilds/KVRT/latest/full/KVRT.exe
> （推荐理由：绿色版、最新病毒库）
> 大蜘蛛：http://free.drweb.ru/download+cureit+free
> （推荐理由：扫描快、一次下载只能用1周，更新病毒库）
> 火绒安全软件：https://www.huorong.cn
> 360杀毒：http://sd.360.cn/download_center.html
### 3. 病毒动态：
>CVERC-国家计算机病毒应急处理中心：http://www.cverc.org.cn
>微步在线威胁情报社区：https://x.threatbook.cn
>火绒安全论坛：http://bbs.huorong.cn/forum-59-1.html
>爱毒霸社区：http://bbs.duba.net
>腾讯电脑管家：http://bbs.guanjia.qq.com/forum-2-1.html
### 4. 在线病毒扫描网站：
>http://www.virscan.org //多引擎在线病毒扫描网 v1.02，当前支持 41 款杀毒引擎
>https://habo.qq.com //腾讯哈勃分析系统
>https://virusscan.jotti.org //Jotti恶意软件扫描系统
>http://www.scanvir.com //针对计算机病毒、手机病毒、可疑文件等进行检测分析
### 5. webshell查杀：
> D盾_Web查杀：http://www.d99net.net/index.asp
> 河马webshell查杀：http://www.shellpub.com
> 深信服Webshell网站后门检测工具：http://edr.sangfor.com.cn/backdoor_detection.html
> Safe3：http://www.uusec.com/webshell.zip
### 6. 威胁情报
>微步在线威胁情报社区：https://x.threatbook.cn
>VenusEye威胁情报中心:https://www.venuseye.com.cn
>奇安信威胁情报中心:https://ti.qianxin.com
>天际友盟的情报https://redqueen.tj-un.com/IntelHome.html


## 参考文档
1. [Windows环境黑客入侵应急与排查](https://blog.csdn.net/wxh0000mm/article/details/102948296)
2. [应急响应实战笔记，一个安全工程师的自我修养。](https://github.com/Bypass007/Emergency-Response-Notes)

