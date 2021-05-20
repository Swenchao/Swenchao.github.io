---
title: Hive DDL数据定义
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-18 20:21:38
password:
summary: Hive 数据定义
tags:
- Hive
categories:
- 大数据
---

# Hive

## DDL 数据定义

### 创建数据库

1. 创建一个数据库，数据库在 HDFS 上的默认存储路径是/user/hive/warehouse

```sql
hive (default)> create database db_hive;
```

2. 避免要创建的数据库已经存在错误，增加 if not exists 判断。(标准写法)

```sql
hive (default)> create database db_hive;
——>
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Database db_hive already exists

hive (default)> create database if not exists db_hive;
```

3. 创建一个数据库，指定数据库在 HDFS 上存放的位置

```sql
hive (default)> create database db_hive2 location '/db_hive2.db';
```

![](4-1.png)

### 查询数据库

#### 显示数据库

1. 显示数据库

```sql
hive> show databases;
——>
OK
database_name
db_hive
db_hive2
default
Time taken: 0.04 seconds, Fetched: 3 row(s)
```

2. 过滤显示查询的数据库

```sql
hive> show databases like 'db_hive\*';
——>
OK
db_hive
db_hive_2
```

#### 查看数据库详情

1. 显示数据库信息

```sql
hive> desc database db_hive;
——>
OK
db_name comment location        owner_name      owner_type      parameters
db_hive         hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db    user_test  USER
Time taken: 0.075 seconds, Fetched: 1 row(s)
```

2. 显示数据库详细信息——extended

```sql
hive> desc database extended db_hive;
——>
OK
db_name comment location        owner_name      owner_type      parameters
db_hive         hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db    user_test  USER
Time taken: 0.086 seconds, Fetched: 1 row(s)
```

#### 切换当前数据库

```sql
hive (default)> use db_hive;
```

### 修改数据库

用户可以使用 ALTER DATABASE 命令为某个数据库的 DBPROPERTIES 设置键-值对属性值，来描述这个数据库的属性信息。数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置。

```sql
hive (default)> alter database db_hive set dbproperties('createtime'='20201111');
```

在 hive 中查看修改结果

```sql
hive (db_hive)> desc database extended db_hive;
——>
OK
db_name comment location        owner_name      owner_type      parameters
db_hive         hdfs://hadoop102:9000/user/hive/warehouse/db_hive.db    user_test     USER    {createtime=20201111}
Time taken: 0.152 seconds, Fetched: 1 row(s)
```

### 删除数据库

1. 删除空数据库

```sql
hive(db_hive)> drop database db_hive2;
```

2. 如果删除的数据库不存在，最好采用 if exists 判断数据库是否存在

```sql
hive(db_hive)> drop database db_hive;
——>
FAILED: SemanticException [Error 10072]: Database does not exist: db_hive

hive> drop database if exists db_hive2;
```

3. 如果数据库不为空，可以采用 cascade 命令，强制删除

```sql
hive(db_hive)> drop database db_hive;
——>
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database db_hive is not empty. One or more tables exist.)

hive> drop database db_hive cascade;
```

### 创建表

1. 建表语法

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name, col_name, ...)
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
[ROW FORMAT row_format]
[STORED AS file_format]
[LOCATION hdfs_path]
```

2. 字段解释说明

(1) CREATE TABLE：创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；可以用 IF NOT EXISTS 选项来忽略这个异常。

(2) EXTERNAL：关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径(LOCATION)，Hive 创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。

(3) COMMENT：为表和列添加注释。

(4)PARTITIONED BY：创建分区表

(5) CLUSTERED BY：创建分桶表

(6) SORTED BY：不常用

**order by | sort by | distribute by**

在 hive 中都有排序和聚集的作用，然而，它们在执行时所启动的 MR 却各不相同。

order by：会对所给的全部数据进行全局排序，并且只会“叫醒”一个 reducer 干活。它就像一个糊涂蛋一样，不管来多少数据，都只启动一个 reducer 来处理。因此，数据量小还可以，但数据量一旦变大 order by 就会变得异常吃力，甚至“罢工”。

sort by：是局部排序。相比 order by 的懒惰糊涂，sort by 正好相反，它不但非常勤快，而且具备分身功能。sort by 会根据数据量的大小启动一到多个 reducer 来干活，并且，它会在进入 reduce 之前为每个 reducer 都产生一个排序文件。这样的好处是提高了全局排序的效率。

distribute by：控制 map 结果的分发，它会将具有相同字段的 map 输出分发到一个 reduce 节点上做处理。即就是，某种情况下，我们需要控制某个特定行到某个 reducer 中，这种操作一般是为后续可能发生的聚集操作做准备。

(7) ROW FORMAT

```
DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]
[MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
| SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
```

用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。如果没有指定 ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，Hive 通过 SerDe 确定表的具体的列的数据。

SerDe 是 Serialize/Deserilize 的简称，目的是用于序列化和反序列化。

(8) STORED AS 指定存储文件类型

常用的存储文件类型：SEQUENCEFILE(二进制序列文件)、TEXTFILE(文本)、RCFILE(列式存储格式文件)。如果文件数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。

(9) LOCATION ：指定表在 HDFS 上的存储位置。

(10) LIKE 允许用户复制现有的表结构，但是不复制数据。

#### 管理表

1. 理论

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive 会(或多或少地)控制着数据的生命周期。Hive 默认情况下会将这些表的数据存储在由配置项 hive.metastore.warehouse.dir (例如，/user/hive/warehouse) 所定义的目录的子目录下。当我们删除一个管理表时，Hive 也会删除这个表中数据。管理表不适合和其他工具共享数据。

2. 案例实操

(1) 普通创建表

```sql
create table if not exists student2(
id int, name string
)
row format delimited fields terminated by '\t'
stored as textfile
location '/user/hive/warehouse/student2';
```

(2) 根据查询结果创建表(查询的结果会添加到新创建的表中)

```sql
create table if not exists student3 as select id, name from student;
```

(3) 根据已经存在的表结构创建表

```sql
create table if not exists student4 like student;
```

(4) 查询表的类型

```sql
hive (default)> desc formatted student2;
——>
...
Table Type: MANAGED_TABLE
...
```

#### 外部表

1. 理论

因为表是外部表，所以 Hive 并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

2. 管理表和外部表的使用场景

每天将收集到的网站日志定期流入 HDFS 文本文件。在外部表(原始日志表)的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过 SELECT+INSERT 进入内部表。

3. 案例实操

分别创建部门和员工外部表，并向表中导入数据。

(1) 原始数据

[dept.txt]

```
10	ACCOUNTING	1700
20	RESEARCH	1800
30	SALES	1900
40	OPERATIONS	1700
```

[emp.txt]

```
7369	SMITH	CLERK	7902	1980-12-17	800.00		20
7499	ALLEN	SALESMAN	7698	1981-2-20	1600.00	300.00	30
7521	WARD	SALESMAN	7698	1981-2-22	1250.00	500.00	30
7566	JONES	MANAGER	7839	1981-4-2	2975.00		20
7654	MARTIN	SALESMAN	7698	1981-9-28	1250.00	1400.00	30
7698	BLAKE	MANAGER	7839	1981-5-1	2850.00		30
7782	CLARK	MANAGER	7839	1981-6-9	2450.00		10
7788	SCOTT	ANALYST	7566	1987-4-19	3000.00		20
7839	KING	PRESIDENT		1981-11-17	5000.00		10
7844	TURNER	SALESMAN	7698	1981-9-8	1500.00	0.00	30
7876	ADAMS	CLERK	7788	1987-5-23	1100.00		20
7900	JAMES	CLERK	7698	1981-12-3	950.00		30
7902	FORD	ANALYST	7566	1981-12-3	3000.00		20
7934	MILLER	CLERK	7782	1982-1-23	1300.00		10
```

(2) 建表语句

创建部门表

```sql
create external table if not exists default.dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

创建员工表

```sql
create external table if not exists default.emp(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

(3) 查看创建的表

```sql
hive (default)> show tables;
——>
OK
tab_name
dept
emp
```

(4) 向外部表中导入数据

导入数据

```sql
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept;

hive (default)> load data local inpath '/opt/module/datas/emp.txt' into table default.emp;
```

查询结果

```sql
hive (default)> select * from emp;
hive (default)> select * from dept;
```

(5) 查看表格式化数据

```sql
hive (default)> desc formatted dept;
——>
...
Table Type: EXTERNAL_TABLE
...
```

#### 管理表与外部表的互相转换

(1) 查询表的类型

```sql
hive (default)> desc formatted student2;
——>
...
Table Type: MANAGED_TABLE
...
```

(2) 修改内部表 student2 为外部表

```sql
hive (default)> alter table student2 set tblproperties('EXTERNAL'='TRUE');
```

(3) 查询表的类型

```sql
hive (default)> desc formatted student2;
——>
...
Table Type: EXTERNAL_TABLE
...
```

(4) 修改外部表 student2 为内部表

```sql
hive (default)> alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

(5) 查询表的类型

```sql
hive (default)> desc formatted student2;
——>
...
Table Type: MANAGED_TABLE
...
```

**注：**('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！

### 分区表

分区表实际上就是对应一个 HDFS 文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。Hive 中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过 WHERE 子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

#### 分区表基本操作

1. 引入分区表(需要根据日期对日志进行管理)

```
/user/hive/warehouse/log_partition/20170702/20170702.log
/user/hive/warehouse/log_partition/20170703/20170703.log
/user/hive/warehouse/log_partition/20170704/20170704.log
```

2. 创建分区表语法

```sql
hive (default)> create table dept_partition(
deptno int, dname string, loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
```

3. 加载数据到分区表中

```sql
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='202009');

hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='202010');

hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='202011’);
```

![](4-2.png)

![](4-3.png)

4. 查询分区表中数据

单分区查询

```sql
hive (default)> select * from dept_partition where month='202009';
```

多分区联合查询

```sql
hive (default)> select * from dept_partition where month='202009'
union
select * from dept_partition where month='202010'
union
select * from dept_partition where month='202011';
——>
_u3.deptno      _u3.dname       _u3.loc _u3.month
10      ACCOUNTING      1700    202009
10      ACCOUNTING      1700    202010
10      ACCOUNTING      1700    202011
20      RESEARCH        1800    202009
20      RESEARCH        1800    202010
20      RESEARCH        1800    202011
30      SALES   1900    202009
30      SALES   1900    202010
30      SALES   1900    202011
40      OPERATIONS      1700    202009
40      OPERATIONS      1700    202010
40      OPERATIONS      1700    202011
```

5. 增加分区

创建单个分区

```sql
hive (default)> alter table dept_partition add partition(month='202008');
```

同时创建多个分区

```sql
hive (default)> alter table dept_partition add partition(month='202007') partition(month='202006');
```

6. 删除分区

删除单个分区

```sql
hive (default)> alter table dept_partition drop partition (month='202008');
```

同时删除多个分区

```sql
hive (default)> alter table dept_partition drop partition (month='202007'), partition (month='202006');
```

7. 查看分区表有多少分区

```sql
hive> show partitions dept_partition;
```

8. 查看分区表结构

```sql
hive> desc formatted dept_partition;
——>
# Partition Information
# col_name  data_type   comment
month string
```

#### 分区表注意事项

1. 创建二级分区表

```sql
hive (default)> create table dept_partition2(
deptno int, dname string, loc string
)
partitioned by (month string, day string)
row format delimited fields terminated by '\t';
```

2. 正常的加载数据

(1) 加载数据到二级分区表中

```sql
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table
default.dept_partition2 partition(month='202009', day='13');
```

(2) 查询分区数据

```sql
hive (default)> select * from dept_partition2 where month='202009' and day='13';
```

3. 把数据直接上传到分区目录上，让分区表和数据产生关联的三种方式

(1) 方式一：上传数据后修复

上传数据

```sql
hive (default)> dfs -mkdir -p
/user/hive/warehouse/dept_partition2/month=202009/day=12;

hive (default)> dfs -put /opt/module/datas/dept.txt /user/hive/warehouse/dept_partition2/month=202009/day=12;
```

查询数据(查询不到刚上传的数据)

```sql
hive (default)> select * from dept_partition2 where month='202009' and day='12';
```

执行修复命令

```
hive> msck repair table dept_partition2;
```

再次查询数据

```sql
hive (default)> select * from dept_partition2 where month='202009' and day='12';
```

(2) 方式二：上传数据后添加分区

上传数据

```sql
hive (default)> dfs -mkdir -p /user/hive/warehouse/ dept_partition2/month=202009/day=11;

hive (default)> dfs -put /opt/module/datas/dept.txt /user/hive/warehouse/dept_partition2/month=202009/day=11;
```

执行添加分区

```sql
hive (default)> alter table dept_partition2 add partition(month='202009',day='11');
```

查询数据

```sql
hive (default)> select * from dept_partition2 where month='202009' and day='11';
```

(3) 方式三：创建文件夹后 load 数据到分区

创建目录

```sql
hive (default)> dfs -mkdir -p /user/hive/warehouse dept_partition2/month=202009/day=10;
```

上传数据

```sql
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table
dept_partition2 partition(month='202009',day='10');
```

查询数据

```sql
hive (default)> select * from dept_partition2 where month='202009' and day='10';
```

### 修改表

#### 重命名表

1. 语法

```sql
ALTER TABLE table_name RENAME TO new_table_name
```

2. 实操案例

```sql
hive (default)> alter table dept_partition2 rename to dept_partition3;
```

#### 增加、修改和删除表分区

详见前面分区表基本操作。

#### 增加/修改/替换列信息

1. 语法

更新列

```sql
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
```

增加和替换列

```sql
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)
```

**注：**ADD 是代表新增一字段，字段位置在所有列后面(partition 列前)，REPLACE 则是表示替换表中所有字段。

2. 实操案例

(1)查询表结构

```sql
hive> desc dept_partition;
```

(2)添加列

```sql
hive (default)> alter table dept_partition add columns(deptdesc string);
```

(3)查询表结构

```sql
hive> desc dept_partition;
```

(4)更新列

```sql
hive (default)> alter table dept_partition change column deptdesc desc int;
```

(5)查询表结构

```sql
hive> desc dept_partition;
```

(6)替换列

```sql
hive (default)> alter table dept_partition replace columns(deptno string, dname string, loc string);
```

(7)查询表结构

```sql
hive> desc dept_partition;
```

### 删除表

```sql
hive (default)> drop table dept_partition;
```
