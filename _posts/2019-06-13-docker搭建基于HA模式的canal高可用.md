---
layout:     post
title:      docker搭建基于HA模式的canal高可用
date:       2019-06-13
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Docker
    - Zookeeper
    - Canal
---
# 一、HA原理图
![HA原理图](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/canal.png?raw=true)

# 二、zookeeper集群搭建
## 1. 搭建服务器地址

ip地址|服务器名称
:----|:----
172.17.3.95|test1
172.17.3.96|test2
172.17.3.101|test3
## 2. docker启动命令

**test1**
```
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zk3 -v /qj/zookeeper/conf/:/conf -v /qj/zookeeper/data:/data --net host zookeeper
```
**test2**
```
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zk1 -v /qj/zookeeper/conf/:/conf -v /qj/zookeeper/data:/data --net host zookeeper
```

**test3**
```
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zk2 -v /qj/zookeeper/conf/:/conf -v /qj/zookeeper/data:/data --net host zookeeper
```
## 3. 配置文件修改
**配置文件zoo.cfg**：宿主机路径/qj/zookeeper/conf

```
clientPort=2181
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
server.1=172.17.3.96:2888:3888
server.2=172.17.3.101:2888:3888
server.3=172.17.3.95:2888:3888
```
**配置文件myid**：宿主机路径/qj/zookeeper/data
设置为数字1、2、3，根据server.id配置

## 4. 查看集群状态

```
cyf@test1:~$ echo stat | nc 172.17.3.101 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.17.3.101:59258[0](queued=0,recved=1,sent=0)
 /172.17.3.101:51456[1](queued=0,recved=9885,sent=9885)
 /172.17.3.96:37598[1](queued=0,recved=34744,sent=34744)
 /172.17.3.96:48640[1](queued=0,recved=14126,sent=16480)

Latency min/avg/max: 0/0/187
Received: 58763
Sent: 61116
Connections: 4
Outstanding: 0
Zxid: 0x40000acaa
Mode: leader
Node count: 31
Proposal sizes last/min/max: 363/32/370
```
# 三、canal-server搭建

## 1. 搭建服务器地址

ip地址|服务器名称
:----|:----
172.17.3.96|test2
172.17.3.101|test3

## 2. docker启动命令

**test2**

```
docker run -d   --name canal -v /qj/canal-server:/home/admin/canal-server  -p 11111:11111 --net host canal/canal-server:v1.1.0
```

**test3**

```
docker run -d   --name canal -v /qj/canal-server:/home/admin/canal-server  -p 11111:11111 --net host canal/canal-server:v1.1.0
```
## 3. 配置文件修改
**配置文件canal.properties**：宿主机路径/qj/canal-server/conf

```
需要修改的地方

canal.id= 2 //两个canal-server不同

canal.zkServers=172.17.3.95:2181,172.17.3.96:2181,172.17.3.101:2181 //zookeeper集群地址

canal.destinations= example,activity_mini 

#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
#canal.instance.global.spring.xml = classpath:spring/file-instance.xml
canal.instance.global.spring.xml = classpath:spring/default-instance.xml
其它地方默认

```
**配置文件instance.properties**：宿主机路径/qj/canal-server/conf/example

```
需要修改的地方

canal.instance.mysql.slaveId=3333 //id号不能与mysql的重复，和其他的instance重复

canal.instance.master.address=172.17.3.96:3316 //链接的mysql地址

# username/password //mysql的用户名密码以及编码格式
canal.instance.dbUsername=root
canal.instance.dbPassword=qj12345678@
canal.instance.connectionCharset=UTF-8

canal.instance.filter.regex=pzhframe_base_organization\\..* //授权的数据库

canal.instance.filter.black.regex= //黑名单
```
[参考链接](https://blog.csdn.net/hackerwin7/article/details/38044327)

# 四、常见问题及解决方案
**1、实现canal同步，mysql必须实现开启二进制**

```
修改mysql的配置文件：mysqld.cnf，添加如下三行即可
[mysqld]
... //忽略
log-bin=mysql-bin
binlog-format=ROW
server_id=1
```
**2、canal不同步的原因**

meta.dat文件记录的参数与数据库有差异，删除meta.dat、h2.mv.db重启canal即可
```
 meta.dat 内容是个json串，大概如下：

{"clientDatas":[{"clientIdentity":{"clientId":1001,"destination":"example","filter":""},"cursor":{"identity":{"slaveId":-1,"sourceAddress":{"address":"10.10.161.84","port":3306}},"postion":{"included":false,"journalName":"mysql-bin.000033","position":5988,"timestamp":1429621093000}}}],"destination":"example"}
clientId 可以参考：canal/logs/example/meta.log
address:主库ip
port:主库端口
journalName : binlog名称。
position:开始同步的位置
timestamp : 延迟的时间（写0会从journalName开头开始同步）。
destination : 实例名（默认应该和当前目录名一致）
注： 实际运行发现，如果指定参数有差异，canal会从journalName的起始位置开始同步。

```
**3、有的destination需要加黑名单**

这个需要开发那边提供

```
需要修改配置文件：instance.properties（每个destination下面有）

canal.instance.filter.regex=pzhframe_base_organization\\..* //数据库名（格式：数据库名\\..*）
# table black regex
canal.instance.filter.black.regex=pzhframe_base_organization\\.eall_department_info //数据库名+表名（格式：数据库名\\.表名）

```
**4、新增destination**

在conf目录下复制一份新的，修改一下名字即可，然后修改配置文件

```
instance.properties：

canal.instance.master.address=172.18.0.1:3316 //同步数据库地址

canal.instance.dbUsername=root
canal.instance.dbPassword=qj12345678@
canal.instance.connectionCharset=UTF-8 //数据库账号密码，及编码

canal.instance.filter.regex= //同步的表
canal.instance.filter.black.regex= //黑名单

canal.properties：

canal.destinations= //将新增的destinations名字添加在这里即可

```
**4、同步位点问题**

删除zookeeper几点钟对应的instance

```
cyf@ali-prod:/qj/script$ docker exec -it zk1 bash
bash-4.4# cd bin
bash-4.4# ./zkCli.sh
Connecting to localhost:2181
2019-03-04 11:38:18,345 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
2019-03-04 11:38:18,349 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=ali-prod
2019-03-04 11:38:18,349 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_181
2019-03-04 11:38:18,353 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2019-03-04 11:38:18,353 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.8-openjdk/jre
2019-03-04 11:38:18,353 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/zookeeper-3.4.13/bin/../build/classes:/zookeeper-3.4.13/bin/../build/lib/*.jar:/zookeeper-3.4.13/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.13/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.13/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.13/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.13/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.13/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.13/bin/../zookeeper-3.4.13.jar:/zookeeper-3.4.13/bin/../src/java/lib/*.jar:/conf:
2019-03-04 11:38:18,353 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2019-03-04 11:38:18,353 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2019-03-04 11:38:18,353 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2019-03-04 11:38:18,354 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2019-03-04 11:38:18,354 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2019-03-04 11:38:18,354 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=4.9.0-8-amd64
2019-03-04 11:38:18,354 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
2019-03-04 11:38:18,354 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
2019-03-04 11:38:18,354 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/zookeeper-3.4.13/bin
2019-03-04 11:38:18,355 [myid:] - INFO  [main:ZooKeeper@442] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@4b85612c
Welcome to ZooKeeper!
2019-03-04 11:38:18,387 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1029] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2019-03-04 11:38:18,473 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@879] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2019-03-04 11:38:18,482 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1303] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x1015398cf540000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 1] ls /otter/canal 
[cluster, destinations]
[zk: localhost:2181(CONNECTED) 2] ls /otter/canal/destinations
[tme_metting_assistant, activity_mini, example]
[zk: localhost:2181(CONNECTED) 4] rmr /otter/canal/destinations/example //删除对应的instance即可
```






