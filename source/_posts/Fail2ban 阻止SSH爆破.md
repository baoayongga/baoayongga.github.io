---
title: Fail2ban 阻止SSH爆破
date: 2019-1-22 08:52:52
tags:
    - Linux
    - Fail2ban
    - SSH
categories: [工具分享]
---

## 简介
Fail2ban可以监视你的系统日志，匹配日志的错误信息执行相应的屏蔽动作。

<!-- more -->

## 安装
```
yum -y install epel-release

yum -y install fail2ban
```


## 配置规则
Fail2ban配置文件在 `/etc/fail2ban`

`jail.conf`防御配置文件

```
vi /etc/fail2ban/jail.local[DEFAULT]
ignoreip = 127.0.0.1/8        #IP白名单，白名单中的IP不会屏蔽，可以用（,）分隔填写多个ip
bantime  = 86400              #屏蔽时间-秒
findtime = 600                #设置多长时间（秒）内超过 maxretry 限制次数即被封锁
maxretry = 3                  #最大尝试次数

#这里banaction需要firewallcmd-ipset
banaction = firewallcmd-ipset #屏蔽IP所使用的方法，firewalld，iptables

action = %(action_mwl)s       #动作参数

[sshd]
enabled = true                # 开启防护，false关闭
filter  = sshd                #过滤规则名称，对应 filter.d 目录下的 sshd.conf
port    = 22                  #对应的端口
action = %(action_mwl)s       #动作参数
logpath = /var/log/secure     #监控的日志路径
```
## 使用
`systemctl start fail2ban`启动`Fail2ban`

同一个IP，在10分钟内，如果连续超过3次错误，则使用Firewalld Ban掉该IP。

输入：`fail2ban-client status sshd` 查看被Ban掉的IP。