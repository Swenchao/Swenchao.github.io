---
title: Hive安装配置
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-06 21:37:17
password:
summary: Hive 安装配置
tags:
- Hive
categories:
- 大数据
---

# Hive

## Hive安装

### Hive安装地址

1．Hive官网地址

[http://hive.apache.org/](http://hive.apache.org/)

2．文档查看地址

[https://cwiki.apache.org/confluence/display/Hive/GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)

3．下载地址

[http://archive.apache.org/dist/hive/](http://archive.apache.org/dist/hive/)

4．github地址

[https://github.com/apache/hive](https://github.com/apache/hive)

### Hive安装部署

#### Hive安装及配置

1. 把 apache-hive-1.2.1-bin.tar.gz 上传到虚拟机 /opt/software 目录下

2. 解压 apache-hive-1.2.1-bin.tar.gz 到 /opt/module/ 目录下面

```shell
[user_test@hadoop102 software]$ tar -zxvf apache-hive-1.2.1-bin.tar.gz -C /opt/module/
```

3. 修改 apache-hive-1.2.1-bin.tar.gz的名称为hive

```shell
[user_test@hadoop102 module]$ mv apache-hive-1.2.1-bin/ hive
```

4. 修改 /opt/module/hive/conf 目录下的 hive-env.sh.template 名称为 hive-env.sh

```shell
[user_test@hadoop102 conf]$ mv hive-env.sh.template hive-env.sh
```

5. 配置 hive-env.sh 文件
	
(1) 配置HADOOP_HOME路径

export HADOOP_HOME=/opt/module/hadoop-2.7.2

(2) 配置HIVE_CONF_DIR路径

export HIVE_CONF_DIR=/opt/module/hive/conf

#### Hadoop集群配置

1. 必须启动hdfs和yarn

```shell
[user_test@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
[user_test@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```

2. 在HDFS上创建 /tmp 和 /user/hive/warehouse 两个目录并修改他们的同组权限可写

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir /tmp
[user_test@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir -p /user/hive/warehouse

[user_test@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /tmp
[user_test@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /user/hive/warehouse
```

3. Hive基本操作

(1) 启动hive

```shell
[user_test@hadoop102 hive]$ bin/hive
```

(2) 查看数据库

```sql
hive> show databases;
```

(3) 打开默认数据库

```sql
hive> use default;
```

(4) 显示default数据库中的表

```sql
hive> show tables;
```

(5) 创建一张表

```sql
hive> create table student(id int, name string);
```

(6) 显示数据库中有几张表

```sql
hive> show tables;
```

(7) 查看表的结构

```sql
hive> desc student;
```

(8) 向表中插入数据

```sql
hive> insert into student values(1, "zhangsan");
hive> insert into student values(2, "lisi");
```

(9) 查询表中数据

```sql
hive> select * from student;
```

(10) 退出hive

```sql
hive> quit;
```

### 将本地文件导入Hive案例

需求

将本地 /opt/module/datas/student.txt 这个目录下的数据导入到 hive 的 student(id int, name string)表中。

1. 数据准备

在/opt/module/datas这个目录下准备数据

(1) 在/opt/module/目录下创建datas

```shell
[user_test@hadoop102 module]$ mkdir datas
```

（2）在/opt/module/datas/目录下创建student.txt文件并添加数据

```shell
[user_test@hadoop102 datas]$ touch student.txt
[user_test@hadoop102 datas]$ vi student.txt
——>
1001    zhangsan
1002    lisi
1003    wangwu
```

**注意:**学号和姓名中间以 tab 键间隔。

2. Hive实际操作

(1) 启动hive

```shell
[user_test@hadoop102 hive]$ bin/hive
```

(2) 显示数据库

```sql
hive> show databases;
```

(3) 使用 default 数据库

```sql
hive> use default;
```

(4) 显示 default 数据库中的表

```sql
hive> show tables;
```

(5) 删除已创建的 student 表

```sql
hive> drop table student;
```

(6) 创建 student 表, 并声明文件分隔符’\t’

```sql
hive> create table student(id int, name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
```

(7) 加载 /opt/module/datas/student.txt 文件到student数据库表中。

```sql
hive> load data local inpath '/opt/module/datas/student.txt' into table student;
```

也可以直接加载 hdfs 的数据

```sql
hive> load data inpath '/student.txt' into table student;
```

上面就是从 hdfs 文件加载数据到 hive

**注意：**以上两种方式虽然最终结果一样，但是从本地上传后，本地的数据源文件还在；从 hdfs 上传后，根目录下不再有 student.txt 文件。其实这个过程就是修改了hdfs上文件的源信息，将其地址进行了修改。

(8) Hive查询结果

```sql
hive> select * from student;
——>
OK
1001    zhangsan
1002    lisi
1003    wangwu
Time taken: 0.083 seconds, Fetched: 3 row(s)
```

3. 遇到的问题

再打开一个客户端窗口启动hive，会产生 java.sql.SQLException 异常。

```java
Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException:
 Unable to instantiate
 org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
        at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:522)
        at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:677)
        at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
Caused by: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
        at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1523)
        at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.<init>(RetryingMetaStoreClient.java:86)
        at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:132)
        at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:104)
        at org.apache.hadoop.hive.ql.metadata.Hive.createMetaStoreClient(Hive.java:3005)
        at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:3024)
        at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:503)
...
```

原因是，Metastore 默认存储在自带的 derby 数据库中，所以只能开一个，因此开两个会报错。修改使用 MySQL 存储 Metastore 可以解决此问题;

### MySql安装

#### 安装包准备

1. 查看mysql是否安装，如果安装了，卸载mysql

执行以下步骤需切换 root，不然可能权限不够

(1) 查看

```shell
[root@hadoop102 桌面]# rpm -qa|grep mysql
```

若执行以上命令什么都没有出来，则说明还未安装，不用进行下面的卸载步骤。若有一些信息，说明已经安装，要执行下面卸载步骤。

(2) 卸载

```shell
[root@hadoop102 桌面]# rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64
```

**--nodeps 删除依赖进行卸载**

2. 解压mysql-libs.zip文件到当前目录

```shell
[root@hadoop102 software]# unzip mysql-libs.zip
[root@hadoop102 software]# ls
——>
mysql-libs.zip
mysql-libs
```

3. 进入到mysql-libs文件夹下

```shell
[root@hadoop102 mysql-libs]# ll
——>
总用量 76048
-rw-r--r--. 1 root root 18509960 3月  26 2015 MySQL-client-5.6.24-1.el6.x86_64.rpm
-rw-r--r--. 1 root root  3575135 12月  1 2013 mysql-connector-java-5.1.27.tar.gz
-rw-r--r--. 1 root root 55782196 3月  26 2015 MySQL-server-5.6.24-1.el6.x86_64.rpm
```

#### 安装MySql服务器

1. 安装mysql服务端

```shell
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
```

2. 查看产生的随机密码

```shell
[root@hadoop102 mysql-libs]# cat /root/.mysql_secret
——>
# The random password set for the root user at Sun Nov  1 20:10:18 2020 (local time): N5AQmsyBqnsWhzFk
```

3. 查看mysql状态

```shell
[root@hadoop102 mysql-libs]# service mysql status
```

4. 启动mysql

```shell
[root@hadoop102 mysql-libs]# service mysql start
——>
Starting MySQL. SUCCESS!
```

出现以上语句，说明启动成功

#### 安装MySql客户端

1. 安装mysql客户端

```shell
[root@hadoop102 mysql-libs]# rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
```

2. 连接mysql

```shell
[root@hadoop102 mysql-libs]# mysql -uroot -pN5AQmsyBqnsWhzFk
```

**-u后面是用户名，-p后面是刚才随机生成的密码**

3. 修改密码

```mysql
mysql> SET PASSWORD=PASSWORD('123456');
```

4. 退出mysql

```mysql
mysql> exit
```

**注：若以上步骤中出现错误，其原因有很多种，最简单暴力的方法有时往往是最有效的——卸载干净重新装（以下步骤）**

1. centos7 默认安装的是 mariadb，需要先卸载 mariadb，先查看是否安装 mariadb

```shell
rpm -qa | grep mariadb
```

如果找到，则使用下面命令删除，如删除 mariadb-libs-5.5.35-3.el7.x86_64**

```shell
rpm -e --nodeps mariadb-libs-5.5.35-3.el7.x86_64
```

2. 查找以前是否安装有mysql，使用下面命令（跟上面相同），若有则进行卸载：

```shell
rpm -qa|grep -i mysql
——>
MySQL-server-5.6.24-1.el6.x86_64
MySQL-client-5.6.24-1.el6.x86_64
```

如果显示有如上包则说明已安装mysql

3. 如果已安装，则需要删除已安装的数据库，使用以下命令来删除数据库

删除命令：rpm -e --nodeps 包名

```shell
rpm -ev mysql-4.1.12-3.RHEL4.1
```

4. 删除老版本mysql的开发头文件和库

```shell
rm -fr /usr/lib/mysql
rm -fr /usr/include/mysql
```

5. 卸载后 /var/lib/mysql 中的数据及 /etc/my.cnf 不会删除，如果确定没用后就手工删除

```shell
rm -f /etc/my.cnf
rm -fr /var/lib/mysql
```

#### MySql中user表中主机配置

配置只要是root用户+密码，在任何主机上都能登录MySQL数据库。

1. 进入mysql

```shell
[root@hadoop102 mysql-libs]# mysql -uroot -p123456
```

2. 显示数据库

```mysql
mysql> show databases;
```

3. 使用mysql数据库

```mysql
mysql> use mysql;
```

4. 展示mysql数据库中的所有表

```mysql
mysql> show tables;
——>
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
28 rows in set (0.00 sec)
```

5. 展示user表的结构

```mysql
mysql> desc user;
```

6. 查询user表

```mysql
mysql> select User, Host, Password from user;
——>
+------+-----------+-------------------------------------------+
| User | Host      | Password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| root | hadoop102 | *2C084B5A757F38D3807BBF835422112631ED67AF |
| root | 127.0.0.1 | *2C084B5A757F38D3807BBF835422112631ED67AF |
| root | ::1       | *2C084B5A757F38D3807BBF835422112631ED67AF |
+------+-----------+-------------------------------------------+
4 rows in set (0.10 sec)
```

**因为此时远程登陆会出现问题，所以要配一个如主机登录：需要以下步骤**

7. 修改user表，把Host表内容修改为%

```mysql
mysql> update user set host='%' where host='localhost';
```

8. 删除root用户的其他host

```mysql
mysql> delete from user where Host='hadoop102';
mysql> delete from user where Host='127.0.0.1';
mysql> delete from user where Host='::1';
```

9. 刷新

```mysql
mysql> flush privileges;
```

10. 退出

```mysql
mysql> quit;
```

### Hive元数据配置到MySql

#### 驱动拷贝

1. 在 /opt/software/mysql-libs 目录下解压 mysql-connector-java-5.1.27.tar.gz 驱动包

```shell
[root@hadoop102 mysql-libs]# tar -zxvf mysql-connector-java-5.1.27.tar.gz
```

2. 拷贝 /opt/software/mysql-libs/mysql-connector-java-5.1.27 目录下的 mysql-connector-java-5.1.27-bin.jar 到 /opt/module/hive/lib/

```shell
[root@hadoop102 mysql-connector-java-5.1.27]# cp mysql-connector-java-5.1.27-bin.jar /opt/module/hive/lib/
```

#### 配置Metastore到MySql

1. 在/opt/module/hive/conf目录下创建一个hive-site.xml

```shell
[user_test@hadoop102 conf]$ touch hive-site.xml
[user_test@hadoop102 conf]$ vi hive-site.xml
```

2. 根据官方文档配置参数，拷贝数据到hive-site.xml文件中

[https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin](https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin)

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://hadoop102:3306/metastore?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>root</value>
	  <description>username to use against metastore database</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>123456</value>
	  <description>password to use against metastore database</description>
	</property>
</configuration>
```

配完上面的再去看表就看不到之前debry中的数据

3. 配置完毕后，如果启动hive异常，可以重新启动虚拟机。（重启后，别忘了启动hadoop集群）

#### 多窗口启动Hive测试

之前在同一服务器上不能同时打开两个hive，但是在不同服务器上可以同时启动多个（因为，在不同服务器上启动hive时，他会在每个服务器的 ./ 目录下重新创建 derbylog 和 metastore_da，相互之间互不相通）

1. 先启动MySQL

```shell
[user_test@hadoop102 mysql-libs]$ mysql -uroot -p123456
```

查看有几个数据库

```mysql
mysql> show databases;
——>
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql             |
| performance_schema |
| test               |
+--------------------+
```

2. 再次打开多个窗口，分别启动hive

```shell
[user_test@hadoop102 hive]$ bin/hive
```

3．启动hive后，回到MySQL窗口查看数据库，显示增加了metastore数据库

```mysql
mysql> show databases;
——>
+--------------------+
| Database           |
+--------------------+
| information_schema |
| metastore          |
| mysql             |
| performance_schema |
| test               |
+--------------------+
```

### HiveJDBC访问

hive可以像mysql一样在项目中写sql进行一些操作，但是太慢了，所以一般不会用。

#### 启动hiveserver2服务

```shell
[user_test@hadoop102 hive]$ bin/hiveserver2
```

#### 启动beeline

```shel
[user_test@hadoop102 hive]$ bin/beeline
——>
Beeline version 1.2.1 by Apache Hive
beeline>
```

#### 连接hiveserver2

```shell
beeline> !connect jdbc:hive2://hadoop102:10000
——>
Connecting to jdbc:hive2://hadoop102:10000
Enter username for jdbc:hive2://hadoop102:10000: user_test（安装过程中使用的用户名）
——>
Enter password for jdbc:hive2://hadoop102:10000: （设置密码，则输入回车；没有设置就直接回车）
——>
Connected to: Apache Hive (version 1.2.1)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://hadoop102:10000> 
```

显示已有数据库

```shell
0: jdbc:hive2://hadoop102:10000> show databases;
——>
+----------------+--+
| database_name  |
+----------------+--+
| default        |
+----------------+--+
```

### Hive常用交互命令

```shell
[user_test@hadoop102 hive]$ bin/hive -help
——>
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the console)
```

1. "-e"不进入hive的交互窗口执行sql语句

```shell
[user_test@hadoop102 hive]$ bin/hive -e "select id from student;"
```

2. "-f"执行脚本中sql语句

(1) 在/opt/module/datas目录下创建hivef.sql文件

```shell
[user_test@hadoop102 datas]$ touch hivef.sql
```

文件中写入正确的sql语句

```sql
select *from student;
```

(2) 执行文件中的sql语句

```shell
[user_test@hadoop102 hive]$ bin/hive -f /opt/module/datas/hivef.sql
```

(3) 执行文件中的sql语句并将结果写入文件中

```shell
[user_test@hadoop102 hive]$ bin/hive -f /opt/module/datas/hivef.sql  > /opt/module/datas/hive_result.txt
```

### Hive其他命令操作

1. 退出hive窗口：

```sql
hive(default)> exit;

hive(default)> quit;
```

在新版的hive中没区别了，在以前的版本是有的：

exit:先隐性提交数据，再退出；

quit:不提交数据，退出；

2. 在hive cli命令窗口中查看hdfs文件系统

```sql
hive(default)> dfs -ls /;
```

3. 在hive cli命令窗口中查看本地文件系统

```sql
hive(default)> ! ls /opt/module/datas;
```

4. 查看在hive中输入的所有历史命令

(1) 进入到当前用户的根目录/root或/home/user_test

(2) 查看.hivehistory文件

```shell
[user_test@hadoop102 ~]$ cat .hivehistory
```

### Hive常见属性配置

#### Hive数据仓库位置配置

1. Default数据仓库的最原始位置是在hdfs上的：/user/hive/warehouse路径下。

2. 在仓库目录下，没有对默认的数据库default创建文件夹。如果某张表属于default数据库，直接在数据仓库目录下创建一个文件夹。

3. 修改default数据仓库原始位置（将hive-default.xml.template如下配置信息拷贝到hive-site.xml文件中）。

```xml
<property>
	<name>hive.metastore.warehouse.dir</name>
	<value>/user/hive/warehouse</value>
	<description>location of default database for the warehouse</description>
</property>
```

配置同组用户有执行权限

```shell
bin/hdfs dfs -chmod g+w /user/hive/warehouse
```

#### 查询后信息显示配置

1. 在hive-site.xml文件中添加如下配置信息，就可以实现显示当前数据库，以及查询表的头信息配置。

```xml
<property>
	<name>hive.cli.print.header</name>
	<value>true</value>
</property>

<property>
	<name>hive.cli.print.current.db</name>
	<value>true</value>
</property>
```

2. 重新启动hive，对比配置前后差异。

（1）配置前，如图6-2所示

```sql
hive> select * from student;
——>
1001    zhangsan
1002    lisi
1003    wangwu
```

（2）配置后，如图6-3所示

```sql
hive (default)> select * from student;
——>
student.id      student.name
1001    zhangsan
1002    lisi
1003    wangwu
```

#### Hive运行日志信息配置

1. Hive的log默认存放在/tmp/atguigu/hive.log目录下（当前用户名下）

2. 修改hive的log存放日志到/opt/module/hive/logs

(1) 修改/opt/module/hive/conf/hive-log4j.properties.template文件名称为hive-log4j.properties

```shell
[user_test@hadoop102 conf]$ pwd
——>
/opt/module/hive/conf

[user_test@hadoop102 conf]$ mv hive-log4j.properties.template hive-log4j.properties
```

(2) 在hive-log4j.properties文件中修改log存放位置

```properties
hive.log.dir=/opt/module/hive/logs
```

#### 参数配置方式

hadoop修改配置的途径：\*-site.xml文件、程序中的classpath下的set方法、代码中configuration.set进行修改

1. 查看当前所有的配置信息（最大任务数）

```sql
hive> set mapred.reduce.tasks;
——>
mapred.reduce.tasks=-1
```

2. 参数的配置三种方式

(1) 配置文件方式

默认配置文件：hive-default.xml

用户自定义配置文件：hive-site.xml

**注：**用户自定义配置会覆盖默认配置。另外，Hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，Hive的配置会覆盖Hadoop的配置。配置文件的设定对本机启动的所有Hive进程都有效。

(2) 命令行参数方式

启动Hive时，可以在命令行添加 "-hiveconf param=value" 来设定参数，例如：

```shell
[user_test@hadoop103 hive]$ bin/hive -hiveconf mapred.reduce.tasks=10;
```

**注：**仅对本次hive启动有效

查看参数设置：

```sql
hive (default)> set mapred.reduce.tasks;
```

(3) 参数声明方式

可以在HQL中使用SET关键字设定参数，例如：

```sql
hive (default)> set mapred.reduce.tasks=100;
```

**注：**仅对本次hive启动有效。

查看参数设置

```sql
hive (default)> set mapred.reduce.tasks;
```

上述三种设定方式的优先级依次递增。即配置文件 < 命令行参数 < 参数声明。注意某些系统级的参数，例如 log4j 相关的设定，必须用前两种方式设定，因为那些参数的读取在会话建立以前已经完成了。
