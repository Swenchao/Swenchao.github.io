---
title: Hive DML数据操作
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-23 14:35:57
password:
summary: DML数据操作
tags:
- Hive
categories:
- 大数据
---

# Hive

## DML 数据操作

### 数据导入

#### 向表中装载数据(Load)

1. 语法

```sql
hive> load data [local] inpath '/opt/module/datas/student.txt' overwrite | into table student [partition (partcol1=val1,…)];
```

(1) load data:表示加载数据

(2) local:表示从本地加载数据到 hive 表；否则从 HDFS 加载数据到 hive 表

(3) inpath:表示加载数据的路径

(4) overwrite:表示覆盖表中已有数据，否则表示追加

(5) into table:表示加载到哪张表

(6) student:表示具体的表

(7) partition:表示上传到指定分区

2. 实操案例

(0) 创建一张表

```sql
hive (default)> create table student(id string, name string) row format delimited fields terminated by '\t';
```

(1) 加载本地文件到 hive

```sql
hive (default)> load data local inpath '/opt/module/datas/student.txt' into table default.student;
```

(2) 加载 HDFS 文件到 hive 中

上传文件到 HDFS

```sql
hive (default)> dfs -put /opt/module/datas/student.txt /user/user_test/hive;
```

加载 HDFS 上数据

```sql
hive (default)> load data inpath '/user/user_test/hive/student.txt' into table default.student;
```

(3) 加载数据覆盖表中已有的数据

上传文件到 HDFS

```sql
hive (default)> dfs -put /opt/module/datas/student.txt /user/user_test/hive;
```

加载数据覆盖表中已有的数据

```sql
hive (default)> load data inpath '/user/user_test/hive/student.txt' overwrite into table default.student;
```

#### 通过查询语句向表中插入数据(Insert)

1. 创建一张分区表

```sql
hive (default)> create table student(id int, name string) partitioned by (month string) row format delimited fields terminated by '\t';
```

2. 基本插入数据

```sql
hive (default)> insert into table student partition(month='202009') values(1,'wangwu');
```

3. 基本模式插入(根据单张表查询结果)

```sql
hive (default)> insert overwrite table student partition (month='202008') select id, name from student where month='202009';
```

4. 多插入模式(根据多张表查询结果)

```sql
hive (default)> from student
               insert overwrite table student partition(month='202007')
               select id, name where month='202009'
               insert overwrite table student partition(month='202006')
               select id, name where month='202009';
```

#### 查询语句中创建表并加载数据(As Select)

详见上面创建表部分。

根据查询结果创建表(查询的结果会添加到新创建的表中)

```sql
create table if not exists student3
as select id, name from student;
```

#### 创建表时通过 Location 指定加载数据路径

5. 创建表，并指定在 hdfs 上的位置

```sql
hive (default)> create table if not exists student5(
id int, name string
)
row format delimited fields terminated by '\t'
location '/user/hive/warehouse/student5';
```

6. 上传数据到 hdfs 上

```sql
hive (default)> dfs -put /opt/module/datas/student.txt
/user/hive/warehouse/student5;
```

7. 查询数据

```sql
hive (default)> select * from student5;
```

#### Import 数据到指定 Hive 表中

**注：**先用 export 导出后，再将数据导入。

```sql
hive (default)> import table student2 partition(month='202009') from
'/user/hive/warehouse/export/student';
```

### 数据导出

#### Insert 导出

1. 将查询的结果导出到本地

```sql
hive (default)> insert overwrite local directory '/opt/module/datas/export/student'
select * from student;
```

2. 将查询的结果格式化导出到本地

```sql
hive(default)>insert overwrite local directory '/opt/module/datas/export/student1'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' select * from student;
```

3. 将查询的结果导出到 HDFS 上(没有 local)

```sql
hive (default)> insert overwrite directory '/user/atguigu/student2' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' select * from student;
```

#### Hadoop 命令导出到本地

```sql
hive (default)> dfs -get /user/hive/warehouse/student/month=202009/000000_0 /opt/module/datas/export/student3.txt;
```

#### Hive Shell 命令导出

基本语法：(hive -f/-e 执行语句或者脚本 > file)

```shell
[atguigu@hadoop102 hive]\$ bin/hive -e 'select * from default.student;' > /opt/module/datas/export/student4.txt;
```

#### Export 导出到 HDFS 上

```sql
hive(default)> export table default.student to
'/user/hive/warehouse/export/student';
```

### 清除表中数据(Truncate)

**注：**Truncate 只能删除管理表，不能删除外部表中数据

```sql
hive (default)> truncate table student;
```
