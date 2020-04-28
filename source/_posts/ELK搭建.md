---
title: ELK学习笔记(一)
date: 2020-04-27 23:56:52
tags: 
	- elk
	- 日志分析
categories: [学习笔记]
---

****

## 基础环境

1. Centos7

2. 关闭Firewalld

   ```
   systemctl stop firewalld
   systemctl disable firewalld
   ```
<!-- more -->

3. 关闭Selinux

![selinux](https://na1r.oss-cn-beijing.aliyuncs.com/20200427165220.png)

4. 192.168.208.137部署Kibana、ES

5. 192.168.208.138部署Logstash



JDK的二进制安装

1. Jdk1.8二进制包下载路径`http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`

2. 解压到对应安装目录`/usr/local/`

 

安装命令

```
cd /usr/local/

tar -zxf jdk-8u201-linux-x64.tar.gz
```

配置Java环境变量

```
vi /etc/profile

export JAVA_HOME=/usr/local/jdk1.8.0_201/

export PATH=$PATH:$JAVA_HOME/bin

export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
```

使配置生效

```
source /etc/profile
```

验证环境变量

```
java -version
```

验证环境变量

![验证环境变量](https://na1r.oss-cn-beijing.aliyuncs.com/20200427185318.png)

## Kibana

1. 下载Kibana二进制包

2. 解压到/usr/local完成安装

Kibana安装脚本

```
cd /usr/local

tar -zxf kibana-6.6.0-linux-x86_64.tar.gz
```

修改Kibana配置

`/usr/local/kibana-6.6.0-linux-x86_64/config/kibana.yml`

```
server.port: 5601

server.host: "0.0.0.0"

#elasticsearch.url: "http://localhost:9200"

#elasticsearch.username: "user"

#elasticsearch.password: "pass"
```

### Kibana的启动和访问

1. 前台启动Kibana：/usr/local/kibana-6.6.0-linux-x86_64/bin/kibana

2. 后台启动Kibana：nohup /usr/local/kibana-6.6.0-linux-x86_64/bin/kibana >/tmp/kibana.log 2>/tmp/kibana.log &

3. 访问Kibana，需要开放5601端口

   ![访问](https://na1r.oss-cn-beijing.aliyuncs.com/20200427190728.png)

   看到这里就表示Kibana安装成功了

### Kibana的安全说明

1. 默认无密码，也是谁都能够访问

2. 如果使用云厂商，可以在安全组控制某个IP的访问

3. 建议借用Nginx实现用户名密码登录

Kibana借用Nginx实现认证

1. Kibana监听在127.0.0.1

2. 部署Nginx，使用Nginx来转发

### Nginx编译安装

```
yum install -y lrzsz wget gcc gcc-c++ make pcre pcre-devel zlib zlib-devel
cd /usr/local/
wget 'http://nginx.org/download/nginx-1.18.0.tar.gz'
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/usr/local/nginx-1.18.0 && make && make install
```

### Nginx环境变量设置

```
vi /etc/profile
export PATH=$PATH:/usr/local/nginx-1.18.0/sbin/
source /etc/profile
```

### Nginx限制源IP访问

```
vi /usr/local/nginx-1.18.0/conf/nginx.conf
```

![Nginx限制源IP访问](https://na1r.oss-cn-beijing.aliyuncs.com/20200427200429.png)

### 查看访问日志

1. /usr/local/nginx-1.18.0/logs/access.log

2. 如果被拒绝了可以在日志里找到源IP

### 修改Kibana配置

`/usr/local/kibana-6.6.0-linux-x86_64/config/kibana.yml`

![127.0.0.1](https://na1r.oss-cn-beijing.aliyuncs.com/20200427203136.png)

### 重启kibana

`ps auxfww|grep kibana`

![重启kibana](https://na1r.oss-cn-beijing.aliyuncs.com/20200427203424.png)

### 重启nginx

```
nginx -s reload
```



### Nginx使用用户名密码

```
vi /usr/local/nginx-1.18.0/conf/nginx.conf
```

![密码文件](https://na1r.oss-cn-beijing.aliyuncs.com/20200427204125.png)

```
touch htpasswd
printf "username:$(openssl passwd -1 userpwd)\n" >/usr/local/nginx-1.18.0/conf/htpasswd
```

只需要修改username以及userpwd即可

![用户名密码](https://na1r.oss-cn-beijing.aliyuncs.com/20200427201840.png)

### 重启nginx

```
nginx -s reload
```

![访问](https://na1r.oss-cn-beijing.aliyuncs.com/20200427202011.png)

## Elasticsearch部署

1. 解压到对应目录完成安装/usr/local/

2. Elasticsearch无法用root启动

```
cd /usr/local/
tar -zxf elasticsearch-6.6.0.tar.gz
```

### Elasticsearch配置

`vi /usr/local/elasticsearch-6.6.0/config/elasticsearch.yml`

```
path.data: /usr/local/elasticsearch-6.6.0/data

path.logs: /usr/local/elasticsearch-6.6.0/logs

network.host: 127.0.0.1

http.port: 9200
```

**在生产环境中，一般是采用集群的方式搭建，不建议节点既有资格成为主节点还要存储数据，压力相当大**

![目录](https://na1r.oss-cn-beijing.aliyuncs.com/20200427213101.png)

### JVM调优

`vi elasticsearch-6.6.0/config/jvm.options`

![jvm.options](https://na1r.oss-cn-beijing.aliyuncs.com/20200427205445.png)

 \#默认为1g，建议修改为内存50%,不然容易出现OOM, 由于虚拟机环境配置较低采用128M

### Elasticsearch的启动

需要普通用户启动

```
useradd -s /sbin/nologin elk
chown -R elk:elk /usr/local/elasticsearch-6.6.0/
su - elk -s /bin/bash
/usr/local/elasticsearch-6.6.0/bin/elasticsearch -d
```

### 是否启动正常

![启动正常](https://na1r.oss-cn-beijing.aliyuncs.com/20200427210100.png)

### Elasticsearch注意事项

1. Elasticsearch如果要跨机器通讯，需要监听在真实网卡上

2. 监听在真实网卡需要调整系统参数才能正常启动

### Elasticsearch监听在非127.0.0.1

1. 监听在0.0.0.0或者内网地址

`vi /usr/local/elasticsearch-6.6.0/config/elasticsearch.yml`

![修改](https://na1r.oss-cn-beijing.aliyuncs.com/20200427210642.png)

### 最大文件打开数

root 权限下

`vi /etc/security/limits.conf`

```
* - nofile 65536
```



![最大文件打开数](https://na1r.oss-cn-beijing.aliyuncs.com/20200427211554.png)

#### 重启Elasticsearch

`ps auxfww`

![杀掉进程](https://na1r.oss-cn-beijing.aliyuncs.com/image-20200427210922011.png)

`/usr/local/elasticsearch-6.6.0/bin/elasticsearch -d`

### 最大打开进程数

root权限

`vi /etc/security/limits.d/20-nproc.conf`

```
* - nproc 4096
```

![4096](https://na1r.oss-cn-beijing.aliyuncs.com/20200427212416.png)

### 内核参数

`vi /etc/sysctl.conf`

```
vm.max_map_count = 262144
```

![内核参数](https://na1r.oss-cn-beijing.aliyuncs.com/20200427212712.png)

`sysctl -p`

修改完大退重新登陆

切换到elk账户

`su - elk -s /bin/bash`

```
/usr/local/elasticsearch-6.6.0/bin/elasticsearch -d
```

### Elasticsearch监听网卡

1. 模拟环境，建议监听在127.0.0.1

2. 云服务器，9200和9300公网入口限制安全组

3. 自建机房，监听在内网网卡，监听在公网容易被入侵

![访问9200](https://na1r.oss-cn-beijing.aliyuncs.com/20200427213628.png)

## Elasticsearch操作

###Elasticsearch的概念

1. 索引 ->类似于Mysql中的数据库

2. 类型 ->类似于Mysql中的数据表

3. 文档 ->存储数据

### Elasticsearch数据操作

1. curl操作Elasticsearch

![curl操作](https://na1r.oss-cn-beijing.aliyuncs.com/20200427213956.png)

2. Kibana操作Elasticsearch

![kibana操作](https://na1r.oss-cn-beijing.aliyuncs.com/20200427214250.png)

### 索引

#### 创建索引

`PUT /na1r`

![创建索引](https://na1r.oss-cn-beijing.aliyuncs.com/20200427214430.png)

#### 获取全部索引

`GET /_cat/indices?v`

![全部索引](https://na1r.oss-cn-beijing.aliyuncs.com/20200427214747.png)

#### 删除索引

`DELETE /na1r`

![删除索引](https://na1r.oss-cn-beijing.aliyuncs.com/20200427214926.png)

### Elasticsearch增删改查

#### 增

![插入数据](https://na1r.oss-cn-beijing.aliyuncs.com/20200427215356.png)

#### 查

`GET /na1r/users/1`

![查1](https://na1r.oss-cn-beijing.aliyuncs.com/20200427215555.png)

`GET /na1r/_search?q=*`

![查2](https://na1r.oss-cn-beijing.aliyuncs.com/20200427215726.png)

#### 改，覆盖

![覆盖](https://na1r.oss-cn-beijing.aliyuncs.com/20200427215915.png)

![改2](https://na1r.oss-cn-beijing.aliyuncs.com/20200427220055.png)

#### 删

`DELETE /na1r/users/1`

![删1](https://na1r.oss-cn-beijing.aliyuncs.com/20200427220150.png)

#### 修改某个字段

不覆盖

使用`doc`标注修改的内容

![修改doc](https://na1r.oss-cn-beijing.aliyuncs.com/20200427220644.png)

![更改结果](https://na1r.oss-cn-beijing.aliyuncs.com/20200427220723.png)



#### 修改所有

修改前

![修改前](https://na1r.oss-cn-beijing.aliyuncs.com/20200427220959.png)

修改后

![1](https://na1r.oss-cn-beijing.aliyuncs.com/20200427221406.png)

![修改后](https://na1r.oss-cn-beijing.aliyuncs.com/20200427221438.png)

#### 增加字段

![增加](https://na1r.oss-cn-beijing.aliyuncs.com/20200427221611.png)

![增加后](https://na1r.oss-cn-beijing.aliyuncs.com/20200427221640.png)

## Logstash部署

### Logstash安装

192.168.208.138部署Logstash

需要Java环境

```
cd /usr/local/
tar -zxf logstash-6.6.0.tar.gz
```

### JVM配置文件

由于模拟环境配置较低

`vi /usr/local/logstash-6.6.0/config/jvm.options`

![jvm配置](https://na1r.oss-cn-beijing.aliyuncs.com/20200428180805.png)

### logstash详情

Logstash的功能分为接收数据、解析过滤并转换数据、输出数据三个部分，

对应的插件依次是input插件、filter插件、output插件，其中，filter插件是可选的

### 常用的

**input**

-  -file：读取一个文件，这个读取功能有点类似于Linux的tail命令，一行一行的实时读取

- -syslog：监听系统514端口的syslog messages，并使用RFC3164格式进行解析

-  -redis：Logstash可以从redis服务器读取数据，redis类似一个消息缓存组件

- -kafka：从kafka服务器读取数据，与logstash架构配合使用一般用在数据量较大的场景。kafka可作数据的缓冲和存储

- -filebeat：文本日志收集器，logstash可以接收filebeat发送过来的数据。

**filter**

filter插件用于数据的过滤、解析和格式化，也就是将非结构化的数据解析成结构化的、可查询的标准化数据。常见filter插件如下：

- -grok：grok是logstash最重要的插件，可解析并结构化任意数据，支持正则，并提供了很多内置的规则和模板可供使用。

- -mutate：此插件提供了丰富的基础类型数据处理能力。包括类型转换、字符串处理和字段处理

- -date：用来转换日志记录中的时间字符串

- -GeoIP：此插件可以根据ip地址提供对应的地域信息，包括国别，省市，经纬度等，对于可视化地图和区域统计非常有用

**output**

output插件用于数据的输出，一个logstash事件可以穿过多个output，直到所有的output处理完毕，这个事件才结束。输出插件常见的如下：

- -elasticsearch：发送数据到elasticsearch

- -file：发送数据到文件

- -redis：发送数据到redis中。redis插件可在input和output插件中

- -kafka：发送数据到kafka中

### Logstash简单的配置

`vi /usr/local/logstash-6.6.0/config/logstash.conf`

```
input{
  stdin{}
}
output{
  stdout{
    codec=>rubydebug
  }
}
```

Logstash启动

**优化启动速度**

`yum install haveged -y `

`systemctl enable haveged`

`systemctl start haveged`

**前台启动**

`/usr/local/logstash-6.6.0/bin/logstash -f /usr/local/logstash-6.6.0/config/logstash.conf`

**后台启动**

`nohup /usr/local/logstash-6.6.0/bin/logstash -f /usr/local/logstash-6.6.0/config/logstash.conf >/tmp/logstash.log 2>/tmp/logstash.log &`

启动后测试一下

![测试](https://na1r.oss-cn-beijing.aliyuncs.com/20200428182142.png)

输入：na1r

输出：

{
       "message" => "na1r",
    "@timestamp" => 2020-04-28T10:20:47.091Z,
      "@version" => "1",
          "host" => "localhost.localdomain"
}

Logstash读取日志

读取登录日志看看

`tail -f /var/log/secure `

`vi /usr/local/logstash-6.6.0/config/logstash.conf`

```
input {
  file {
    path => "/var/log/secure"
  }
}
output{
  stdout{
    codec=>rubydebug
  }
}
```

启动

`/usr/local/logstash-6.6.0/bin/logstash -f /usr/local/logstash-6.6.0/config/logstash.conf`

![输出日志](https://na1r.oss-cn-beijing.aliyuncs.com/20200428182857.png)

### Logstash读取日志发送到ES

用nginx日志试一下

本机安装nginx

#### Nginx编译安装

```
yum install -y lrzsz wget gcc gcc-c++ make pcre pcre-devel zlib zlib-devel
cd /usr/local/
wget 'http://nginx.org/download/nginx-1.18.0.tar.gz'
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/usr/local/nginx-1.18.0 && make && make install
```

#### Nginx环境变量设置

```
vi /etc/profile
export PATH=$PATH:/usr/local/nginx-1.18.0/sbin/
source /etc/profile
```

`vi /usr/local/nginx-1.18.0/conf/nginx.conf`

![去掉注释](https://na1r.oss-cn-beijing.aliyuncs.com/20200428184421.png)

### 开启nginx

**注：开启nginx可能会出现报错**

![报错](https://na1r.oss-cn-beijing.aliyuncs.com/20200428184906.png)

touch相应的文件然后

`nginx -t`

![nginx -t](https://na1r.oss-cn-beijing.aliyuncs.com/20200428185027.png)

`/usr/local/nginx-1.18.0/sbin/nginx -c /usr/local/nginx-1.18.0/conf/nginx.conf`

**启动**

`nginx -s reload`

**访问**

![访问](https://na1r.oss-cn-beijing.aliyuncs.com/20200428184810.png)

**telnet**

看看es是不是通的，下图就表示通的。

![通的](https://na1r.oss-cn-beijing.aliyuncs.com/20200428185332.png)

#### 修改logstash.conf

`vi /usr/local/logstash-6.6.0/config/logstash.conf`

```
input {
  file {
    path => "/usr/local/nginx-1.18.0/logs/access.log"
  }
}
output {
  elasticsearch {
    hosts => ["http://192.168.208.137:9200"]
  }
}
```

![配置2](https://na1r.oss-cn-beijing.aliyuncs.com/20200428185527.png)

#### 启动logstash

`/usr/local/logstash-6.6.0/bin/logstash -f /usr/local/logstash-6.6.0/config/logstash.conf`

#### 看一下nginx日志

![nginx日志](https://na1r.oss-cn-beijing.aliyuncs.com/20200428185827.png)

#### 访问ES

![访问ES](https://na1r.oss-cn-beijing.aliyuncs.com/20200428190250.png)

#### Logstash收集日志

1. 日志文件需要有新日志产生

2. Logstash跟Elasticsearch要能通讯

## Kibana查询数据

`GET /logstash-xx/_search?q=*`

![查询](https://na1r.oss-cn-beijing.aliyuncs.com/20200428190547.png)

![数据](https://na1r.oss-cn-beijing.aliyuncs.com/20200428190627.png)

**创建索引查看日志**

![创建索引查看日志](https://na1r.oss-cn-beijing.aliyuncs.com/20200428190953.png)

![选中时间戳](https://na1r.oss-cn-beijing.aliyuncs.com/20200428191103.png)

**创建完成后点击Discover**

![Discover](https://na1r.oss-cn-beijing.aliyuncs.com/20200428191237.png)

日志就比较直观的显示出来了

**选择自动刷新时间**

![](https://na1r.oss-cn-beijing.aliyuncs.com/20200428191437.png)

可以访问nginx多造一些日志看看

### Kibana简单查询

1. 根据字段查询：message: "_msearch"

   ![message](https://na1r.oss-cn-beijing.aliyuncs.com/20200428191758.png)

2. 根据字段查询：选中查询

![针对查询](https://na1r.oss-cn-beijing.aliyuncs.com/20200428191919.png)

**elk基本入门笔记，后续针对正则表达式学习笔记再整理一下，还有Filebeat以及Redis等**