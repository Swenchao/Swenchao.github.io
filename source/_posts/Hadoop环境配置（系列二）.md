---
title: Hadoop环境配置（系列二）
top: true
cover: true
toc: true
mathjax: true
date: 2020-07-30 18:45:37
password:
summary: Hadoop基本环境搭建
tags:
- Hadoop
categories:
- 大数据
---

Hadoop基本环境搭建（虚拟机配置、jdk以及hadoop安装）

# Hadoop环境配置

## 虚拟机环境

1. 克隆虚拟机

2. 修改克隆虚拟机的静态IP

3. 修改主机名

4. 关闭防火墙

5. 创建普通用户（以后使用多使用普通用户，不然有些操作会很危险）

6. 配置普通用户具有root权限

7. 在/opt目录下创建文件夹

	（1）在/opt目录下创建module、software文件夹（module：存放解压缩内容；software：存放压缩包）
		sudo mkdir module
		sudo mkdir software
	（2）修改module、software文件夹的所有者 
		sudo chown user:user（用户名：组名） module/ software/

## JDK

1. 上传jdk压缩文件到software文件夹下
2. 将jdk压缩包解压缩到module文件夹下
	tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt/module/
3. 配置jdk环境变量

	（1）进入到jdk目录下，获取jdk路径
		pwd
	（2）编辑/etc/profile
		vim sudo /etc/profile  （若vim不可用，使用vi）
	（3）载文件末尾加上java路径
		#JAVA_HOME
		export JAVA_HOME=/opt/module/jdk1.8.0_144
		export PATH=$PATH:$JAVA_HOME/bin
	（4）保存退出
		:wq
	（5）使修改后文件生效
		source /etc/profile
	（6）测试是否配置成功
		java -version
		（会显示 java version "1.8.0_144"... 这就说明成功了）

**注：**centos7之后自带了jdk，可以看下版本，版本低于1.7就要卸掉重装了。可用 which java来看下是不是我们所配置的那个路径，如果不是则说明是系统自带的

## Hadoop

1. 环境配置

跟jdk类似——先上传压缩包，然后解压到指定文件夹，然后编辑环境变量（载profile文件末尾加上环境变量）

**hadoop环境变量配置（加在profile文件末尾）：**

```shell
##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

2. 目录结构

![](1.png)

（1）bin目录：存放对Hadoop相关服务（HDFS,YARN）进行操作的脚本
（2）etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
（3）lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
（4）sbin目录：存放启动或停止Hadoop相关服务的脚本
（5）share目录：存放Hadoop的依赖jar包、文档、和官方案例

# 接下来

接下来要做的是，hadoop几种运行模式的尝试（伪分布式模式以及完全分布式模式）

