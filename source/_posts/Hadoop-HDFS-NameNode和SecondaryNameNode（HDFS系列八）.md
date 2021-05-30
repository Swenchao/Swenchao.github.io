---
title: Hadoop-HDFS-NameNode和SecondaryNameNode（HDFS系列八）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-23 21:53:31
password:
summary: NameNode和SecondaryNameNode相关（面试重点）
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## NameNode和SecondaryNameNode（面试开发重点）

### NN和2NN工作机制

**思考：NameNode中的元数据是存储在哪里的？**

首先，我们做个假设，如果存储在NameNode节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的FsImage。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦NameNode节点断电，就会产生数据丢失。因此，引入Edits文件(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits中。这样，一旦NameNode节点断电，可以通过FsImage和Edits的合并，合成元数据。

但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage和Edits的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，专门用于FsImage和Edits的合并。

NN和2NN工作机制，如下：

![](1.png)

1. 第一阶段：NameNode启动

（1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

（2）客户端对元数据进行增删改的请求。

（3）NameNode记录操作日志，更新滚动日志。

（4）NameNode在内存中对数据进行增删改。

**之所以先更新日志再进行操作，是因为如果先操作数据，未写日志突然断电，那么数据操作有可能丢失，无法还原。**

2. 第二阶段：Secondary NameNode工作

（1）Secondary NameNode 询问NameNode是否需要 CheckPoint。直接带回NameNode是否检查结果。
CheckPoint是一个检查点，检测是否需要合并日志文件和 fsimage 然后再序列化到 fsimge 中。

（2）若触发条件满足了，则 Secondary NameNode 请求执行CheckPoint。

（3）NameNode滚动正在编写的Edits日志，避免在合并的时候没法继续编写操作（001为原来的，然后新的日志先写到002中）。

（4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。

（5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。

（6）生成新的镜像文件 fsimage.chkpoint。

（7）拷贝 fsimage.chkpoint 到 NameNode。

（8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage（现在情况是002和新的镜像文件fsimage合起来就是当前内存中最新的内容）。

**NN和2NN工作机制详解**

Fsimage：NameNode内存中元数据序列化后形成的文件。

Edits：记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）。

NameNode 启动时，先滚动Edits并生成一个空的 edits.inprogress，然后加载Edits和Fsimage 到内存中，此时 NameNode 内存就持有最新的元数据信息。

Client 开始对NameNode 发送元数据的增删改的请求，这些请求的操作首先会被记录到edits.inprogress 中（查询元数据的操作不会被记录在Edits中，因为查询操作不会更改元数据信息），如果此时 NameNode 挂掉，重启后会从 Edits 中读取元数据的信息。然后，NameNode 会在内存中执行元数据的增删改的操作。

由于 Edits 中记录的操作会越来越多，Edits文件会越来越大，导致NameNode在启动加载Edits时会很慢，所以需要对 Edits 和 Fsimage 进行合并（所谓合并，就是将Edits和Fsimage加载到内存中，照着Edits中的操作一步步执行，最终形成新的Fsimage）。

SecondaryNameNode的作用就是帮助NameNode进行Edits和Fsimage的合并工作。
SecondaryNameNode首先会询问NameNode是否需要CheckPoint（触发CheckPoint需要满足两个条件中的任意一个，定时时间到和Edits中数据写满了）。直接带回NameNode是否检查结果。SecondaryNameNode执行CheckPoint操作，首先会让NameNode滚动Edits并生成一个空的edits.inprogress，滚动Edits的目的是给Edits打个标记，以后所有新的操作都写入edits.inprogress，其他未合并的Edits和Fsimage会拷贝到SecondaryNameNode的本地，然后将拷贝的Edits和Fsimage加载到内存中进行合并，生成fsimage.chkpoint，然后将fsimage.chkpoint拷贝给NameNode，重命名为Fsimage后替换掉原来的Fsimage。

NameNode在启动时就只需要加载之前未合并的Edits和Fsimage即可，因为合并过的Edits中的元数据信息已经被记录在Fsimage中。

### Fsimage和Edits解析

1. 概念

![](2.png)

2. oiv查看Fsimage文件

（1）查看oiv和oev命令

```shell
[user_test@hadoop102 current]$ hdfs
——>
...
oiv		apply the offline fsimage viewer to an fsimage
oev		pply the offline edits viewer to an edits file
...
```

（2）基本语法

```shell
hdfs oiv -p （要将 fsimage 转化成的格式） -i （镜像文件） -o （转换后文件输出路径）
```

（3）案例实操（其中文件名称可能有区别）

```shell
[user_test@hadoop102 current]$ pwd
——>
/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current

[user_test@hadoop102 current]$ hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-2.7.2/fsimage.xml

[user_test@hadoop102 current]$ cat /opt/module/hadoop-2.7.2/fsimage.xml
```

将显示的xml文件内容拷贝到idea中创建的xml文件中，并格式化。部分显示结果如下（代码信息会有所区别）。

```xml
<inode>
	<id>16386</id>
	<type>DIRECTORY</type>
	<name>user</name>
	<mtime>1512722284477</mtime>
	<permission>user_test:supergroup:rwxr-xr-x</permission>
	<nsquota>-1</nsquota>
	<dsquota>-1</dsquota>
</inode>
<inode>
	<id>16387</id>
	<type>DIRECTORY</type>
	<name>user_test</name>
	<mtime>1512790549080</mtime>
	<permission>user_test:supergroup:rwxr-xr-x</permission>
	<nsquota>-1</nsquota>
	<dsquota>-1</dsquota>
</inode>
<inode>
	<id>16389</id>
	<type>FILE</type>
	<name>wc.input</name>
	<replication>3</replication>
	<mtime>1512722322219</mtime>
	<atime>1512722321610</atime>
	<perferredBlockSize>134217728</perferredBlockSize>
	<permission>user_test:supergroup:rw-r--r--</permission>
	<blocks>
		<block>
			<id>1073741825</id>
			<genstamp>1001</genstamp>
			<numBytes>59</numBytes>
		</block>
	</blocks>
</inode >
```

**思考：可以看出，Fsimage中没有记录块所对应DataNode，为什么？**

在集群启动后，要求DataNode上报数据块信息，并间隔一段时间后再次上报。（datanode运行机制会详细说）

3. oev查看Edits文件

（1）基本语法

```shell
hdfs oev -p （要将 fsimage 转化成的格式） -i （镜像文件） -o （转换后文件输出路径）
```

（2）案例实操

```shell
[user_test@hadoop102 current]$ hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-2.7.2/edits.xml

[user_test@hadoop102 current]$ cat /opt/module/hadoop-2.7.2/edits.xml
```
将显示的xml文件内容拷贝到 idea 中创建的xml文件中，并格式化。显示结果如下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<EDITS>
	<EDITS_VERSION>-63</EDITS_VERSION>
	<RECORD>
		<OPCODE>OP_START_LOG_SEGMENT</OPCODE>
		<DATA>
			<TXID>129</TXID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ADD</OPCODE>
		<DATA>
			<TXID>130</TXID>
			<LENGTH>0</LENGTH>
			<INODEID>16407</INODEID>
			<PATH>/hello7.txt</PATH>
			<REPLICATION>2</REPLICATION>
			<MTIME>1512943607866</MTIME>
			<ATIME>1512943607866</ATIME>
			<BLOCKSIZE>134217728</BLOCKSIZE>
			<CLIENT_NAME>DFSClient_NONMAPREDUCE_-1544295051_1</CLIENT_NAME>
			<CLIENT_MACHINE>192.168.1.5</CLIENT_MACHINE>
			<OVERWRITE>true</OVERWRITE>
			<PERMISSION_STATUS>
				<USERNAME>user_test</USERNAME>
				<GROUPNAME>supergroup</GROUPNAME>
				<MODE>420</MODE>
			</PERMISSION_STATUS>
			<RPC_CLIENTID>908eafd4-9aec-4288-96f1-e8011d181561</RPC_CLIENTID>
			<RPC_CALLID>0</RPC_CALLID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ALLOCATE_BLOCK_ID</OPCODE>
		<DATA>
			<TXID>131</TXID>
			<BLOCK_ID>1073741839</BLOCK_ID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_SET_GENSTAMP_V2</OPCODE>
		<DATA>
			<TXID>132</TXID>
			<GENSTAMPV2>1016</GENSTAMPV2>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ADD_BLOCK</OPCODE>
		<DATA>
			<TXID>133</TXID>
			<PATH>/hello7.txt</PATH>
			<BLOCK>
				<BLOCK_ID>1073741839</BLOCK_ID>
				<NUM_BYTES>0</NUM_BYTES>
				<GENSTAMP>1016</GENSTAMP>
			</BLOCK>
			<RPC_CLIENTID></RPC_CLIENTID>
			<RPC_CALLID>-2</RPC_CALLID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_CLOSE</OPCODE>
		<DATA>
			<TXID>134</TXID>
			<LENGTH>0</LENGTH>
			<INODEID>0</INODEID>
			<PATH>/hello7.txt</PATH>
			<REPLICATION>2</REPLICATION>
			<MTIME>1512943608761</MTIME>
			<ATIME>1512943607866</ATIME>
			<BLOCKSIZE>134217728</BLOCKSIZE>
			<CLIENT_NAME></CLIENT_NAME>
			<CLIENT_MACHINE></CLIENT_MACHINE>
			<OVERWRITE>false</OVERWRITE>
			<BLOCK>
				<BLOCK_ID>1073741839</BLOCK_ID>
				<NUM_BYTES>25</NUM_BYTES>
				<GENSTAMP>1016</GENSTAMP>
			</BLOCK>
			<PERMISSION_STATUS>
				<USERNAME>user_test</USERNAME>
				<GROUPNAME>supergroup</GROUPNAME>
				<MODE>420</MODE>
			</PERMISSION_STATUS>
		</DATA>
	</RECORD>
</EDITS >
```

**说明：**

    OP_START_LOG_SEGMENT：日志开始
    OP_MKDIR：创建路径
    OP_ADD：上传文件
    OP_ALLOCATE_BLOCK_ID：分配块 id
    OP_SET_GENSTAMP_V2：创建时间戳
    OP_ADD_BLOCK：添加块信息
    OP_CLOSE：操作结束
    OP_RENAME_OLD：重命名原来复制的文件

**思考：NameNode如何确定下次开机启动的时候合并哪些Edits？**

根据 seen_txid 文件中数据来合并，其中记录了最新的。

# 待续...

CheckPoint时间点设置以及NN故障处理
