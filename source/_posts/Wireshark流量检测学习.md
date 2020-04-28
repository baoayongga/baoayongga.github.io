---
title: Wireshark流量检测学习
date: 2020-03-30 23:35:52
tags:
    - Wireshark
    - 流量
    - 抓包
categories: [学习笔记]
---

## Wireshark入门简介

Wireshark（前称Ethereal）是一个网络封包分析软件。网络封包分析软件的功能是撷取网络封包，并尽可能显示出最为详细的网络封包资料。

<!-- more -->

![1](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/1.png)

### 软件的功能

- 解决网络故障问题
- 溯源网络流量
- 分析网络底层协议

### 相关软件

- 科来网络分析系统
- Httpwatch
- Fiddler
- Sniffer

### 支持平台

- Windows
- Linux
- MacOS

## Wireshark底层框架

![2](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/2.png)

GTK1/2：图形处理工具，处理用户的输入输出。

Core：核心引擎，通过函数调用将其他模块连接在一起，起到联动调用的作用。

Wiretap：此时获取的是一些比特流，通过Wiretap（格式支持引擎）能从抓包文件中读取数据包，支持多种文件格式。

Capture：抓包引擎，利用libpcap/WinPcap底层抓取网络数据包，libpcap/WinPcap提供了通用的抓包接口，能从不同类型的网络接口（包括以太网、令牌环网、ATM网等）获取数据包。

Win-/libpcap：Wireshark抓包时依赖的库文件。

## Wireshark抓包原理

- 本地环境

  - 最基本的抓包方式，不借助其他第三方设备，抓取本地网卡进出网络流量，针对自己网卡不能针对整个局域网

    ![3](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/3.png)

- 集线器环境

  - 同一域，集线器是网络层设备，不学习数据包，广播所有接口（已经很少有这种环境）

- 交换机环境

  - 镜像端口，将数据拷贝一份到一个端口，交换机端口做SAPN端口镜像操作

    ![4](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/4.png)

  - ARP欺骗, 假设没有权限在交换机上做端口镜像技术，因为有MAC地址表，又想获取整个局域网中的流量，窃取到其他PC上的流量。这可以通过著名的ARP攻击软件Cain&Abel实现

    ![5](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/5.png)

  - MAC泛洪

    ![6](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/6.png)

## Wireshark下载

下载地址：https://www.wireshark.org/

## Wireshark 调试功能

### 界面设置

Wireshark运行后，界面如下图所示，包括标题栏、菜单栏、工具栏、数据包过滤栏、数据包列表区、数据包详细区、数据包字节区、数据包统计区。

![7](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/7.png)

点击“捕获”->“选项”如下图所示，设置输入接口和过滤器。

数据包详细区是解析数据包核心区域。数据包包含时间流、原始IP地址和目的IP地址、协议、长度、内容等，当双击它能看到对应的详细信息。

### 列设置

列设置包括默认列表、增加列、修改列、隐藏列、删除列，设置默认即可，下图是选中某行右键“应用为列”增加列。

![8](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/9.png)

![10](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/10.png)

### 时间设置

![11](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/11.png)



也可以右键Time列

### 数据包合并导出

![13](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/13.png)

### 抓包过滤器

![14](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/14.png)

- 类型（Type）
  - host、net、port
- 方向（Dir）
  - src、dst
- 协议（Proto）
  - ether、ip、tcp、udp、http、ftp
- 逻辑运算符
  - `&&` 与
  - `||` 或
  - `!` 非

#### 示例：

抓取源地址为192.168.1.101，目的端口为80的流量

```
src host 192.168.1.101 && dst port 80
```

抓取192.168.1.101和192.168.1.102的流量

```
host 192.168.1.101 || host 192.168.1.102
```

不抓取广播包

```repl
! broadcast
```

过滤IP地址：

```
host 192.168.1.101
src host 192.168.1.101
dst host 192.168.1.101
```

过滤端口：

```
port 80
!port 80
dst port 80
src port 80
```

### 显示过滤器

![15](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/15.png))

#### 比较符：

- `==` 等于
- `!=` 不等于
- `>` 大于
- `<` 小于
- `>=` 大于等于
- `<=` 小于等于

#### 逻辑操作符：

- and 两个条件同时满足
- or 其中一个条件被满足
- xor 有且仅有一个条件被满足
- not 没有条件被满足

#### ip地址：

- ip.addr ip地址
- ip.src 源ip
- ip.dst 目标ip

#### 端口过滤：

- tcp.port
- tcp.srcport
- tcp.dstport
- tcp.flags.syn 过滤包含tcp的syn请求的包
- tcp.flags.ack 过滤包含tcp的ack应答的包

#### 协议过滤：

arp、ip、icmp、udp、tcp、bootp、dns

#### 示例：

过滤IP地址：

```
ip.addr == 192.168.1.101   过滤该地址的包
ip.src == 172.16.1.101  过滤源地址为该地址的包
```

过滤端口：

```
tcp.port == 80 过滤tcp中端口号为80的包
tcp.flags.syn == 1 过滤syn请求为1的包
```

## Wireshark高级功能

### 数据追踪流

主要是将TCP、UDP、SSL数据流进行重组并完整呈现出来

![16](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/16.png)

### 捕获文件属性

![17](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/17.png)

### 专家信息

Wireshark会跟在踪捕获文件中发现的任何异常和其他感兴趣的项目，并将其显示在“专家信息”对话框中。目的是让您更好地了解不常见或值得注意的网络行为，并使新手和专家用户比手动扫描数据包列表更快地发现网络问题。

![18](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/18.png)

![19](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/19.png)

每个专家信息项都有一个严重性级别。从最低到最高使用以下级别。Wireshark使用不同的颜色标记它们，这些颜色显示在括号中：


- 聊天（蓝色）
  - 有关常规工作流程的信息，例如设置了SYN标志的TCP数据包。
- 注意（青色）
  - 值得注意的事件，例如应用程序返回了常见的错误代码，例如HTTP 404。
- 警告（黄色）
  - 警告，例如应用程序返回了异常错误代码，例如连接问题。
- 错误（红色）
  - 严重的问题，例如格式错误的数据包。

### 协议分层统计

![20](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/20.png)



### 网络节点及会话统计

![21](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/21.png)

针对性的去分析，如流量情况等。

### 图表分析

可以用来进行报告图形化流量展示。

![22](https://na1r.oss-cn-beijing.aliyuncs.com/wireshark/22.png)


* 参考文档

[[网络安全自学篇] 十三.Wireshark抓包原理（ARP劫持、MAC泛洪）及数据流追踪和图像抓取（二）](https://blog.csdn.net/Eastmount/article/details/101101829 "[网络安全自学篇] 十三.Wireshark抓包原理（ARP劫持、MAC泛洪）及数据流追踪和图像抓取（二）")