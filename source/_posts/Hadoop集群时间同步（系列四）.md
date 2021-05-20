---
title: 集群时间同步（系列四）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-01 15:18:36
password:
summary: Hadoop搭建集群之后，同步集群时间
tags:
- Hadoop
categories:
- 大数据
---

# 集群时间同步

时间同步的方式：找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，比如，每隔十分钟，同步一次时间。

![](1.png)

## 1. 时间服务器配置（必须root用户）

（1）检查ntp（网络时间协议）是否安装

	[root@hadoop102 桌面]# rpm -qa|grep ntp

出现以下内容说明已安装

    ntp-4.2.6p5-10.el6.centos.x86_64
    fontpackages-filesystem-1.41-1.1.el6.noarch
    ntpdate-4.2.6p5-10.el6.centos.x86_64

（2）修改ntp配置文件

	[root@hadoop102 桌面]# vi /etc/ntp.conf

修改内容

a）授权192.168.1.0-192.168.1.255网段上的所有机器可以从这台机器上查询和同步时间

    #restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
    ——>
    restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

b）集群在局域网中，不使用其他互联网上的时间

    server 0.centos.pool.ntp.org iburst
    server 1.centos.pool.ntp.org iburst
    server 2.centos.pool.ntp.org iburst
    server 3.centos.pool.ntp.org iburst
    ——>
    #server 0.centos.pool.ntp.org iburst
    #server 1.centos.pool.ntp.org iburst
    #server 2.centos.pool.ntp.org iburst
    #server 3.centos.pool.ntp.org iburst

c）当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步（添加）

    server 127.127.1.0
    fudge 127.127.1.0 stratum 10

（3）修改/etc/sysconfig/ntpd 文件

	[root@hadoop102 桌面]# vim /etc/sysconfig/ntpd

增加如下内容（让硬件时间与系统时间一起同步）

	SYNC_HWCLOCK=yes

（4）查看状态并重新启动ntpd服务

	[root@hadoop102 桌面]# service ntpd status
	[root@hadoop102 桌面]# service ntpd start

## 2. 其他机器配置（必须root用户）

（1）在其他机器配置10分钟与时间服务器同步一次

	[root@hadoop103桌面]# crontab -e

编写定时任务：

标准格式：* * * * *  xxxxx（任务）

其中第一个 * 代表每一小时第 * 分钟；第二个代表每一天第 * 个小时；第三个代表每月第 * 天；第四个代表每年第 * 个月；第五个代表每周星期 * ；后面的 xxx 是我们要做的任务。

	*/10 * * * * /usr/sbin/ntpdate hadoop102

这是cronetab定时任务写法，可以搜一下具体的。

（2）修改任意机器时间

	[root@hadoop103桌面]# date -s "2017-9-11 11:11:11"

（3）十分钟后查看机器是否与时间服务器同步

	[root@hadoop103桌面]# date

**注：测试的时候，可调成1分钟**