![hive_logo_medium](assets/hive_logo_medium.jpg)hive 1.2.2版本安装指南

# 一、课前准备

1. 要求安装好hadoop 2.x版本的三节点集群,并配置好JAVA_HOME和HADOOP_HOME两个环境变量。如果还没准备好请参考【三节点hadoop2大数据环境安装教程】，有些同学可能会在hadoop3环境下安装1.2.2，但是在启动hive的时候会报错提示版本不兼容的问题。

# 二、课堂主题

讲解hive1.2.2版本的三种常见安装部署模式

# 三、课堂目标

1. 熟练搭建内嵌式hive环境
2. 熟练搭建本地式hive环境
3. 熟练搭建远程式hive环境
4. mysql数据库的安装.

# 四、知识要点（1个小时）

## 1.准备hive安装包

自行按照1.1教程提示下载hive的安装包**或者使用老师提供的hive安装包.**

### 1.1 下载hive

[下载地址](<https://apache.org/dist/hive/hive-1.2.2/>) 打开下载地址后，如下图点击apache-hive-1.2.2-bin.tar.gz  下载

![1562761296813](assets/1562761296813.png)

### 1.2 上传hvie安装包

基于我们之前的环境安装情况已经可以了解到我们已经在node1上部署了namenode，resourcemanager，secondarynamenode等比较重要的进程；node3上呢我们已经安装了centos的桌面和idea,这两个主要的进程消耗的系统资源比较多，那么接下来我们要安装的hive计划安装在node2节点上,所以我们将hive的安装包通过xhsell中的xftp的工具上传到node2上.

如下图

![1562763876004](assets/1562763876004.png)

如下图,

![1562763811988](assets/1562763811988.png)

安装包上传成功,如下图

![1562763927237](assets/1562763927237.png)

### 1.3 解压hive安装包

```shell
#1.把hive的压缩安装包解压到/opt/bigdata/目录下
[root@node2 ~]# tar -xzvf apache-hive-1.2.2-bin.tar.gz -C /opt/bigdata/   #输入完命令后回车
#2.切换到bigdata目录下
[root@node2 ~]# cd /opt/bigdata/
#3.修改hive安装目录的所属用户和组为hadoop:hadoop
[root@node2 bigdata]# chown -R hadoop:hadoop apache-hive-1.2.2-bin/
#4.修改hive安装目录的读写权限
[root@node2 bigdata]# chmod -R 755  apache-hive-1.2.2-bin/	
```

## 2、内嵌模式

元数据保存在内嵌的derby数据库中，只允许一个会话链接，尝试多个会话链接时会报错

### 2.1 启动hive

```shell
[root@node2 bigdata]# chmod -R 755  apache-hive-1.2.2-bin/
You have new mail in /var/spool/mail/root
#1.修改完hive安装目录的所属用户组和读写权限之后，我们使用su - hadoop命令切换到hadoop用户
[root@node2 bigdata]# su - hadoop
Last login: Mon Jul  8 17:44:54 CST 2019 on pts/0
#2.切换到hive的安装目录
[hadoop@node2 ~]$ cd /opt/bigdata/apache-hive-1.2.2-bin/
#3.我们先试用ll命令列举一下hive安装目录的结构，如下:
[hadoop@node2 apache-hive-1.2.2-bin]$ ll
total 84
drwxr-xr-x 3 hadoop hadoop   119 Jul 11 10:57 bin
drwxr-xr-x 2 hadoop hadoop   212 Jul 11 10:57 conf
-rw-rw-r-- 1 hadoop hadoop 21099 Jul 11 11:29 derby.log
drwxr-xr-x 4 hadoop hadoop    34 Jul 11 10:57 examples
drwxr-xr-x 7 hadoop hadoop    68 Jul 11 10:57 hcatalog
drwxr-xr-x 4 hadoop hadoop  8192 Jul 11 10:57 lib
-rwxr-xr-x 1 hadoop hadoop 24754 Mar 31  2017 LICENSE
-rwxr-xr-x 1 hadoop hadoop   397 Mar 31  2017 NOTICE
-rwxr-xr-x 1 hadoop hadoop  4374 Apr  1  2017 README.txt
-rwxr-xr-x 1 hadoop hadoop  4255 Apr  1  2017 RELEASE_NOTES.txt
drwxr-xr-x 3 hadoop hadoop    23 Jul 11 10:57 scripts
[hadoop@node2 apache-hive-1.2.2-bin]$ ./bin/hive  #启动hive
#请忽略此行的日志提示,不影响hive的执行
ls: cannot access /opt/spark-2.4.3-bin-hadoop2.7/lib/spark-assembly-*.jar: No such file or directory

#3.初始化hive
Logging initialized using configuration in jar:file:/opt/bigdata/apache-hive-1.2.2-bin/lib/hive-common-1.2.2.jar!/hive-log4j.properties
#4.hive命令提示符:hive>
#5.第一次安装先显示下hive中默认有哪些数据库，我们可以看到只有一个default默认的数据库.
hive> show databases;
OK
default
Time taken: 0.56 seconds, Fetched: 1 row(s)
hive>

```

我们将hive启动后，在hive的安装目录下会产生一个文件(derby.log)和一个文件夹(metastore_db),在当前node2的连接窗口上，右键，点击复制ssh渠道打开新的一个node2的ssh连接窗口。

![1562837462267](assets/1562837462267.png)

如下图，在hive的安装目录下会产生一个文件(derby.log，保存hive命令以及执行sql命令过程中的一些后台日志)和一个文件夹(metastore_db,保存在hive中创建的库和表的原信息比如库名称，表名称，表中列名称，列的数据，等元数据)

![1562837654503](assets/1562837654503.png)

同时我们还会发现./bin/hive启动hive的时候会在hdfs 的tmp的目录下创建hvie的目录

```shell
[hadoop@node3 ~]$ hdfs dfs -ls /tmp
Found 2 items
drwx------   - hadoop supergroup          0 2019-07-05 21:03 /tmp/hadoop-yarn
#1.创建了hive的目录
drwx-wx-wx   - hadoop supergroup          0 2019-07-11 17:46 /tmp/hive
[hadoop@node3 ~]$ hdfs dfs -ls /tmp/hive
Found 1 items
drwx------   - hadoop supergroup          0 2019-07-11 17:46 /tmp/hive/hadoop
[hadoop@node3 ~]$ hdfs dfs -ls /tmp/hive/hadoop
Found 1 items
drwx------   - hadoop supergroup          0 2019-07-11 17:46 /tmp/hive/hadoop/ebeadee8-6034-4beb-bc2b-35a632aa31ef
[hadoop@node3 ~]$ 
```

### 2.2 使用hive

#### 2.2.1 创建表 

```shell
hive> create table pokes(foo int,bar string);
OK
Time taken: 0.808 seconds
#1.使用show tables;显示default数据库下的所有表;
hive> show tables;
OK
#2.刚刚创建的表
pokes
Time taken: 0.072 seconds, Fetched: 1 row(s)
hive>
```

#### 2.2.2 插入数据



```shell
hive> insert into pokes(foo,bar) values(1,'first bar');


hive> 
```

另一种的方式是使用load的方式将指定的文件加载到hive表中,参考下面的操作.

```shell
#1.使用hive的load 命令将hive本身提供的示例数据加载到pokes表中
hive> LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
Loading data to table default.pokes
Table default.pokes stats: [numFiles=1, numRows=0, totalSize=5812, rawDataSize=0]
OK
Time taken: 2.269 seconds
hive> select count(*) from pokes;
Query ID = hadoop_20190712141621_989a3ed0-b9cc-4546-8e20-941614789bdc
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Starting Job = job_1562912119783_0001, Tracking URL = http://node1:18088/proxy/application_1562912119783_0001/
Kill Command = /opt/bigdata/hadoop-2.7.3/bin/hadoop job  -kill job_1562912119783_0001
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2019-07-12 14:16:32,167 Stage-1 map = 0%,  reduce = 0%
2019-07-12 14:16:46,940 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 3.65 sec
2019-07-12 14:16:53,221 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 6.13 sec
MapReduce Total cumulative CPU time: 6 seconds 130 msec
Ended Job = job_1562912119783_0001
MapReduce Jobs Launched: 
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 6.13 sec   HDFS Read: 12221 HDFS Write: 4 SUCCESS
Total MapReduce CPU Time Spent: 6 seconds 130 msec
OK
500
Time taken: 33.884 seconds, Fetched: 1 row(s)

```

#### 2.2.3 查询表数据

```shell
#1.我们使用select语句查询三条数据出来看看结果吧
hive> select * from pokes limit 3;
OK
#2.显示结果，欧了，搞定.
238	val_238
86	val_86
311	val_311
Time taken: 0.07 seconds, Fetched: 3 row(s)
hive>
```

#### 2.2.4 创建库

```shell
#1.使用create database kkb;语句创建数据库;
hive>  create database kkb;
OK
Time taken: 0.591 seconds
hive> show databases;
OK
default
#2.刚刚创建的库
kkb
Time taken: 0.015 seconds, Fetched: 2 row(s)
hive>
```



```shell
#1.查看/user/hive/warehouse/
[root@node3 ~]# hdfs dfs -ls /user/hive/warehouse/
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2019-07-12 14:21 /user/hive/warehouse/kkb.db
drwxr-xr-x   - hadoop supergroup          0 2019-07-12 14:15 /user/hive/warehouse/pokes
[root@node3 ~]# 
```

通过我们查看hdfs的hive目录我们，可以看到创建库的时候其实就是在hdfs上创建一个目录作为数据库的表存储目录,然后在对应数据库目录下面存储的就是表数据文件。

### 2.3 hive直接启动报错问题分析

#### 2.3.1 Name node is in safe mode

```shell
[hadoop@node2 apache-hive-1.2.2-bin]$ ./bin/hive
ls: cannot access /opt/spark-2.4.3-bin-hadoop2.7/lib/spark-assembly-*.jar: No such file or directory

Logging initialized using configuration in jar:file:/opt/bigdata/apache-hive-1.2.2-bin/lib/hive-common-1.2.2.jar!/hive-log4j.properties
Exception in thread "main" java.lang.RuntimeException: 
#1.提示Name node处于安全模式,安全模式时hadoop对外是不提供服务的，也就是说hive不能使用hdfs来存储数据.
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create directory /tmp/hive. Name node is in safe mode.
The reported blocks 0 needs additional 6 blocks to reach the threshold 0.9990 of total blocks 6.
The number of live datanodes 0 has reached the minimum number 0. Safe mode will be turned off automatically once the thresholds have been reached.
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1327)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:3895)
	at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:984)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:622)
	at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:616)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2049)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2045)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1698)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2043)

	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:522)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:677)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create directory /tmp/hive. Name node is in safe mode.
The reported blocks 0 needs additional 6 blocks to reach the threshold 0.9990 of total blocks 6.
The number of live datanodes 0 has reached the minimum number 0. Safe mode will be turned off automatically once the thresholds have been reached.
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1327)
	at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.mkdirs(FSNamesystem.java:3895)
	at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.mkdirs(NameNodeRpcServer.java:984)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.mkdirs(ClientNamenodeProtocolServerSideTranslatorPB.java:622)
	at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:616)
	at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:982)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2049)
	at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2045)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1698)
	at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2043)

	at org.apache.hadoop.ipc.Client.call(Client.java:1475)
	at org.apache.hadoop.ipc.Client.call(Client.java:1412)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:229)
	at com.sun.proxy.$Proxy14.mkdirs(Unknown Source)
	at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.mkdirs(ClientNamenodeProtocolTranslatorPB.java:558)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:191)
	at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:102)
	at com.sun.proxy.$Proxy15.mkdirs(Unknown Source)
	at org.apache.hadoop.hdfs.DFSClient.primitiveMkdir(DFSClient.java:3000)
	at org.apache.hadoop.hdfs.DFSClient.mkdirs(DFSClient.java:2970)
	at org.apache.hadoop.hdfs.DistributedFileSystem$21.doCall(DistributedFileSystem.java:1047)
	at org.apache.hadoop.hdfs.DistributedFileSystem$21.doCall(DistributedFileSystem.java:1043)
	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirsInternal(DistributedFileSystem.java:1043)
	at org.apache.hadoop.hdfs.DistributedFileSystem.mkdirs(DistributedFileSystem.java:1036)
	at org.apache.hadoop.hive.ql.exec.Utilities.createDirsWithPermission(Utilities.java:3678)
	at org.apache.hadoop.hive.ql.session.SessionState.createRootHDFSDir(SessionState.java:597)
	at org.apache.hadoop.hive.ql.session.SessionState.createSessionDirs(SessionState.java:554)
	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:508)
	... 8 more
#2.出现此问题时不要着急，我们使用 hadoop dfsadmin -safemode get命令查看集群情况
[hadoop@node2 apache-hive-1.2.2-bin]$ hadoop dfsadmin -safemode get
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.
#3.发现安全模式是开启的，这时我们需要使用命令关闭它
Safe mode is ON
#4.使用hadoop dfsadmin -safemode leave命令关闭安全模式
[hadoop@node2 apache-hive-1.2.2-bin]$ hadoop dfsadmin -safemode leave
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.
#5.安全模式已经关闭
Safe mode is OFF
#6.再次查看安全模式是否已经关闭
[hadoop@node2 apache-hive-1.2.2-bin]$ hadoop dfsadmin -safemode get
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.
#7.安全模式已经关闭
Safe mode is OFF
#8.关闭hadoop安全模式之后我们再次重新启动hive
[hadoop@node2 apache-hive-1.2.2-bin]$ ./bin/hive  #启动hive
#请忽略此行的日志提示,不影响hive的执行
ls: cannot access /opt/spark-2.4.3-bin-hadoop2.7/lib/spark-assembly-*.jar: No such file or directory

#9.初始化hive
Logging initialized using configuration in jar:file:/opt/bigdata/apache-hive-1.2.2-bin/lib/hive-common-1.2.2.jar!/hive-log4j.properties
#10.hive命令提示符:hive>
#11.第一次安装先显示下hive中默认有哪些数据库，我们可以看到只有一个default默认的数据库.
hive> show databases;
OK
default
Time taken: 0.56 seconds, Fetched: 1 row(s)
hive>
```

#### 2.3.2 Execution Error, return code 1 

```shell
hive> LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
Loading data to table default.pokes
Failed with exception Unable to move source file:/opt/bigdata/apache-hive-1.2.2-bin/examples/files/kv1.txt to destination hdfs://node1:9000/user/hive/warehouse/pokes/kv1.txt
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.MoveTask
```

```
hive> LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
Loading data to table default.pokes
Failed with exception Call From node2/192.168.200.12 to node1:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.MoveTask
```

问题分析:

  1.从报错信息中可以看到DataNode节点有异常，导致hive 命令执行失败

解决方法：

  1.这时我们需要重新启动下集群就可以了.

## 3.安装mysql数据库

### 3.1 安装wget工具

```shell
#1.安装wget工具
[root@node2 ~]# yum install -y wget
Loaded plugins: fastestmirror
Determining fastest mirrors
#省略部分日志                                                       
Complete!
#2.下载mysql的yum源安装文件,使用下面的命令就直接下载了安装用的Yum Repository，大概25KB的样子，然后就可以直接yum安装了。
[root@node2 ~]# wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
#省略部分日志

100%[======================================================>] 25,548       120KB/s   in 0.2s   
#省略部分日志
Total wall clock time: 3.2s
Downloaded: 1 files, 25K in 0.2s (120 KB/s)
You have new mail in /var/spool/mail/root
[root@node2 ~]# ll
total 88764
-rw-------. 1 root root     1259 Jul  2 04:54 anaconda-ks.cfg
-rw-r--r--  1 root root 90859180 Jul 10 20:58 apache-hive-1.2.2-bin.tar.gz
-rw-r--r--  1 root root    25548 Apr  7  2017 mysql57-community-release-el7-10.noarch.rpm
#3.安装mysql的yum源配置
[root@node2 ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm 
Loaded plugins: fastestmirror

Installed:
  mysql57-community-release.noarch 0:el7-10                                                     
Complete!
You have new mail in /var/spool/mail/root
[root@node2 ~]# 
```

### 3.2 yum安装MySQL

这步花费的时间比较多，安装完成后就会覆盖掉之前的mariadb。

```shell
[root@node2 ~]# yum -y install mysql-community-server
#这步可能会花些时间，安装完成后就会覆盖掉之前的mariadb。
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.jdcloud.com
 * extras: mirror.jdcloud.com
 * updates: mirror.bit.edu.cn
mysql-connectors-community                                               | 2.5 kB  00:00:00     
#省略部分日志...

#出现Complete!这样的字样提示表示安装成功
Complete!
You have new mail in /var/spool/mail/root
[root@node2 ~]# 
```

### 3.3 启动mysql

```shell
[root@node2 ~]# systemctl start  mysqld.service
[root@node2 ~]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   #状态是active（running）就对了，说明mysql数据库的服务已经成功启动!
   Active: active (running) since Wed 2019-07-17 15:48:44 CST; 1min 57s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 109515 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 109430 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 109518 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─109518 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Jul 17 15:48:31 node2 systemd[1]: Starting MySQL Server...
Jul 17 15:48:44 node2 systemd[1]: Started MySQL Server.
You have new mail in /var/spool/mail/root
[root@node2 ~]# 
```

mysql服务启动后的状态截图如下图:

![1563349921008](assets/1563349921008.png)

### 3.4 登录mysql

此时此刻，我们通过上面的几个步骤已经成功安装完并启动mysql，不过想要登录mysql时，我们需要知道mysql的密码，就在我们安装的过程中mysql已经默认给我们生成了一个root用户的默认密码,通过使用root用户和默认密码可以登录mysql，登录之后我们需要将默认的密码修改掉(为了mysql数据库的安全着想).

```shell
#1.查看默认密码
[root@node2 ~]# grep "password" /var/log/mysqld.log
2019-07-17T07:48:39.339873Z 1 [Note] A temporary password is generated for 
#2.显示默认密码:n8Ae>q70PeHG
root@localhost: n8Ae>q70PeHG
You have new mail in /var/spool/mail/root
#3.使用命令mysql -uroot -p登录mysql
[root@node2 ~]# mysql -uroot -p     # 回车后会提示输入密码
Enter password: #
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.26

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
#3.使用命令show databases;显示所有的数据库
mysql> show databases;
#4.因为我们没有修改默认密码，会提示我们修改密码
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
#6.使用下面命令修改密码为:!Qaz123456
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '!Qaz123456';
#7.设置ok
Query OK, 0 rows affected (0.00 sec)

#8.再次执行show databases;
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> 

```

如下图,默认密码

![1563350508104](assets/1563350508104.png)

注意另一种密码格式

![1565255458277](assets/1565255458277.png)

## 4、本地模式

本地安装模式其实就是把mysql和hive安装在一台机器上而已(这就是本地模式)

（本地安装mysql 替代derby存储元数据,使用mysql进行hive的元数据信息管理）

### 4.1 复制hive配置文件

新打开一个链接node2的xshell窗口，切换到hadoop用户,切换目录到hive的安装目录下的conf目录

```shell
#1.在conf目录下从hive-default.xml.template复制出来一个配置文件hive-site.xml文件
#hive-site.xml文件中配置链接mysql的相关信息
[hadoop@node2 conf]$ cp hive-default.xml.template hive-site.xml
#2.使用vi命令编辑hive-site.xml文件并设置行号,按shift 输入：set number回车发现行号设置成功
[hadoop@node2 conf]$ vi hive-site.xml 

<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?><!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
--><configuration>
  <!-- WARNING!!! This file is auto generated for documentation purposes ONLY! -->
  <!-- WARNING!!! Any changes you make to this file will be ignored by Hive.   -->
  <!-- WARNING!!! You must make your changes in hive-site.xml instead.         -->
:set number
```

如下图，行号设置成功,按照教程继续往下操作

![1563353126756](assets/1563353126756.png)

在此按下shift+: 时 set number的命令就会消失，意味着我们可以接着输入新的命令,那就输入新的命令吧

如下图，:18,3913d 输入后回车，会发现<configuration></configuration>标签中间多余的内容已经删除掉了。

![1563353393303](assets/1563353393303.png)

在标签<configuration></configuration>中填入以下内容

```xml
<property> 
    <!--这个是用于存放hive元数据的目录位置。 -->
  <name>hive.metastore.warehouse.dir</name>  
  <value>/user/hive_remote/warehouse</value>  
</property>  
<!-- 控制hive是否连接一个远程metastore服务器还是开启一个本地客户端jvm，默认是true，Hive0.10已经取消了该配置项；-->
<property>  
  <name>hive.metastore.local</name>  
  <value>true</value>  
</property>  
   <!-- JDBC连接字符串，默认jdbc:derby:;databaseName=metastore_db;create=true；-->
<property>  
  <name>javax.jdo.option.ConnectionURL</name>  
  <value>jdbc:mysql://localhost/hive_remote?createDatabaseIfNotExist=true</value>  
</property>  
   <!--JDBC的driver，默认org.apache.derby.jdbc.EmbeddedDriver； -->
<property>  
  <name>javax.jdo.option.ConnectionDriverName</name>  
  <value>com.mysql.jdbc.Driver</value>  
</property>  
   <!-- username，默认APP；-->
<property>  
  <name>javax.jdo.option.ConnectionUserName</name>  
  <value>root</value>  
</property>  
   <!--password，默认mine；-->
<property>  
  <name>javax.jdo.option.ConnectionPassword</name>  
  <value>!Qaz123456</value>  
</property>  

```

### 4.2 下载mysql驱动包

[下载mysql驱动包](<https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.46>)

![1563354990218](assets/1563354990218.png)



如下图，新打开一个node2的xshell连接，使用xftp将下载的mysql的驱动包上传到root用户的家目录下

![1563354813113](assets/1563354813113.png)

将下载的mysql驱动包复制到hive的lib目录下

```shell
[root@node2 ~]# cp mysql-connector-java-5.1.46.jar  /opt/bigdata/apache-hive-1.2.2-bin/lib/
You have new mail in /var/spool/mail/root
[root@node2 ~]#
```

### 4.3 启动hive

切换到hadoop用户,执行hive命令启动hive

```shell
#1.切换到hive的安装目录，使用ll命令，查看hive的安装目录
[hadoop@node2 apache-hive-1.2.2-bin]$ ll
total 84
drwxr-xr-x 3 hadoop hadoop   119 Jul 11 10:57 bin
drwxr-xr-x 2 hadoop hadoop   233 Jul 17 16:55 conf
-rw-rw-r-- 1 hadoop hadoop 21099 Jul 12 14:15 derby.log
drwxr-xr-x 4 hadoop hadoop    34 Jul 11 10:57 examples
drwxr-xr-x 7 hadoop hadoop    68 Jul 11 10:57 hcatalog
drwxr-xr-x 4 hadoop hadoop  8192 Jul 17 17:18 lib
-rwxr-xr-x 1 hadoop hadoop 24754 Mar 31  2017 LICENSE
drwxrwxr-x 5 hadoop hadoop   133 Jul 12 14:15 metastore_db
-rwxr-xr-x 1 hadoop hadoop   397 Mar 31  2017 NOTICE
-rwxr-xr-x 1 hadoop hadoop  4374 Apr  1  2017 README.txt
-rwxr-xr-x 1 hadoop hadoop  4255 Apr  1  2017 RELEASE_NOTES.txt
drwxr-xr-x 3 hadoop hadoop    23 Jul 11 10:57 scripts
#2.删除掉derby模式生成的日志文件和元数据目录,我们现在使用mysql进行管理元数据信息，所以这里不再需要他们，我们就删除掉即可.
[hadoop@node2 apache-hive-1.2.2-bin]$ rm -rf derby.log 
[hadoop@node2 apache-hive-1.2.2-bin]$ rm -rf metastore_db/
#3.重新启动hive
注意：启动hive失败时看这篇文章：https://blog.csdn.net/qq331948781/article/details/52695822

[hadoop@node2 apache-hive-1.2.2-bin]$ ./bin/hive
ls: cannot access /opt/spark-2.4.3-bin-hadoop2.7/lib/spark-assembly-*.jar: No such file or directory
19/07/17 17:23:15 WARN conf.HiveConf: HiveConf of name hive.metastore.local does not exist

Logging initialized using configuration in jar:file:/opt/bigdata/apache-hive-1.2.2-bin/lib/hive-common-1.2.2.jar!/hive-log4j.properties
Wed Jul 17 17:23:19 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:20 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:20 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:20 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:20 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:20 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:21 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Wed Jul 17 17:23:21 CST 2019 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
#4.启动后我们执行show databases;命令显示所有数据库,hive的本地mysql数据库配置完成.
hive> show databases;
OK
default
Time taken: 0.902 seconds, Fetched: 1 row(s)
#5.退出hive
hive> quit;
#6.在显示下hive安装目录的所有信息，我们发现目录下没有产生新文件,此时的元数据已经存入到mysql数据库中了，继续下面的步骤我们会看到.
[hadoop@node2 apache-hive-1.2.2-bin]$ ll
total 60
drwxr-xr-x 3 hadoop hadoop   119 Jul 11 10:57 bin
drwxr-xr-x 2 hadoop hadoop   233 Jul 17 16:55 conf
drwxr-xr-x 4 hadoop hadoop    34 Jul 11 10:57 examples
drwxr-xr-x 7 hadoop hadoop    68 Jul 11 10:57 hcatalog
drwxr-xr-x 4 hadoop hadoop  8192 Jul 17 17:18 lib
-rwxr-xr-x 1 hadoop hadoop 24754 Mar 31  2017 LICENSE
-rwxr-xr-x 1 hadoop hadoop   397 Mar 31  2017 NOTICE
-rwxr-xr-x 1 hadoop hadoop  4374 Apr  1  2017 README.txt
-rwxr-xr-x 1 hadoop hadoop  4255 Apr  1  2017 RELEASE_NOTES.txt
drwxr-xr-x 3 hadoop hadoop    23 Jul 11 10:57 scripts
[hadoop@node2 apache-hive-1.2.2-bin]$ 
```

### 4.4 查看hive的元数据信息

#### 4.4.1 mysql中的元数据

经过上面的hive配置之后，我们来看下mysql中有什么样的变化呢?，经过我们的多次操作我们已经在xshell中打开了三个连接node2的窗口,如下图,切换到已经登录mysql的窗口，按照图中的提示操作.

![1563355906494](assets/1563355906494.png)

接着上面的操作，我们继续向下探索....

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hive_remote        |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

--1.切换当前数据库为hive_remote,hive_remote数据库就是我们在hive-site.xml文件中配置的默认生成的数据库.
mysql> use hive_remote;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
--2.一下显示的是hive的元数据信息的所有表
mysql> show tables;
+---------------------------+
| Tables_in_hive_remote     |
+---------------------------+
| BUCKETING_COLS            |
| CDS                       |
| COLUMNS_V2                |
| DATABASE_PARAMS           |
| DBS                       |
| FUNCS                     |
| FUNC_RU                   |
| GLOBAL_PRIVS              |
| PARTITIONS                |
| PARTITION_KEYS            |
| PARTITION_KEY_VALS        |
| PARTITION_PARAMS          |
| PART_COL_STATS            |
| ROLES                     |
| SDS                       |
| SD_PARAMS                 |
| SEQUENCE_TABLE            |
| SERDES                    |
| SERDE_PARAMS              |
| SKEWED_COL_NAMES          |
| SKEWED_COL_VALUE_LOC_MAP  |
| SKEWED_STRING_LIST        |
| SKEWED_STRING_LIST_VALUES |
| SKEWED_VALUES             |
| SORT_COLS                 |
| TABLE_PARAMS              |
| TAB_COL_STATS  
| --存储在hive中创建的表的表明  |
| TBLS                      |
| VERSION                   |
+---------------------------+
29 rows in set (0.00 sec)

mysql> 
```



切换到运行hive的xshell窗口，启动hive，并创建表

```sql
[hadoop@node2 apache-hive-1.2.2-bin]$ ./bin/hive

hive> show tables;
OK
Time taken: 0.672 seconds
hive> create table pokes(foo int,bar string);
OK
Time taken: 0.347 seconds
hive> 
```

如下图的操作

![1563356757497](assets/1563356757497.png)

好，此时我们已经在hive中创建了表，那MySQL中有什么样的变化，我们继续探索....

```sql
--1.执行下面的语句，发现我们刚才在hive中创建的表pokes的表名.
mysql> select TBL_NAME from TBLS;
+----------+
| TBL_NAME |
+----------+
| pokes    |
+----------+
1 row in set (0.00 sec)

mysql> 
```

如下图，操作结果

![1563357106757](assets/1563357106757.png)

#### 4.4.2 hdfs中的元数据

```shell
[hadoop@node2 ~]$ hdfs dfs -ls /user
Found 3 items
drwxr-xr-x   - hadoop supergroup          0 2019-07-13 15:00 /user/hadoop
drwxr-xr-x   - hadoop supergroup          0 2019-07-11 17:56 /user/hive
#1.当我们在hive命令行中执行创建表的语句之后，同时会在hdfs上创建/user/hive_remote目录.
drwxr-xr-x   - hadoop supergroup          0 2019-07-17 17:42 /user/hive_remote
[hadoop@node2 ~]$ 
[hadoop@node2 ~]$ 
```

至此我们本地mysql管理hive元数据安装结束.

## 5、远程模式

远程模式的意思就是mysql的安装和hive的安装没有在同一台机器上，和本地模式最大的区别就是在hive-site.xml中，我们需要修改下mysql的ip地址和端口号。但是我们需要注意以下两个问题:

1. hive-site.xml中的mysql安装主机的ip地址和端口号.
2. 访问mysql的用户名的访问权限需要进行设置.

### 5.1 安装远端mysql

继续我们的探索之旅吧，目前我们已经在node2上安装过了mysql，此时我们要进行远程模式的mysql安装，我们选择在node3节点上安装mysql（**参考安装mysql数据库**），然后设置在node2上使用root用户能够访问node3上的mysql的权限。

按照【安装mysql数据库】安装完mysql数据库，修改完初始化密码之后，需要修改数据库的远程访问权限(访问者在远端，访问mysql的操作没有在mysql安装机器上执行sql语句或者mysql命令).

```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '!Qaz123456';
Query OK, 0 rows affected (0.00 sec)

--授权使用root用户名和密码（!Qaz123456）,可以通过任意机器(%符号代替所有机器)访问mysql数据库
mysql> grant all on *.* to root@'%' identified by '!Qaz123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)
--使上一步授权生效
mysql> flush privileges;
```

### 5.2 修改hive-site.xml配置

**因为我们只是改变了mysql的安装位置，hive的安装位置不需要改变，我们只需要将hive安装目录conf目录下的hive-site.xml文件中的mysql的连接ip修改成node3的ip即可，这样就完成了远程模式的配置.**

```
<property>  
  <name>hive.metastore.warehouse.dir</name>  
  <value>/user/hive_remote/warehouse</value>  
</property>  

<property>  
  <name>hive.metastore.local</name>  
  <value>true</value>  
</property>  
   
<property>  
  <name>javax.jdo.option.ConnectionURL</name>  
  <value>jdbc:mysql://192.168.200.13/hive_remote?createDatabaseIfNotExist=true</value>  
</property>  
   
<property>  
  <name>javax.jdo.option.ConnectionDriverName</name>  
  <value>com.mysql.jdbc.Driver</value>  
</property>  
   
<property>  
  <name>javax.jdo.option.ConnectionUserName</name>  
  <value>root</value>  
</property>  
   
<property>  
  <name>javax.jdo.option.ConnectionPassword</name>  
  <value>!Qaz123456</value>  
</property>  
```

### 5.3 启动hive

因为我们只是改变了mysql的安装位置，hive的安装位置不需要改变，我们以同样的方式启动hive.

```shell
#启动hive
[hadoop@node2 apache-hive-1.2.2-bin]$ ./bin/hive

#显示hive表
hive> show tables;
OK
Time taken: 0.713 seconds
hive> 
```

查看mysql数据库中的变化,如下图操作，我们发现存放hive元数据的数据库hive_remote创建成功.

![1563432387019](assets/1563432387019.png)



使用同样的在hive中创建表，在hdfs查看创建的hive表目录

```sql
 create table pokes(foo int,bar string);
```

![1563432986248](assets/1563432986248.png)

![1563432964462](assets/1563432964462.png)