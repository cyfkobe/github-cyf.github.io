---
layout:     post
title:      搭建redis一主一从集群
date:       2019-05-06
author:     cyf
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Redis
    - Docker
---
## 一、利用docker-machine搭建测试环境
### docker-machine
创建两个虚拟机
```shell
docker-machine create -d virtualbox master
docker-machine create -d virtualbox slave
```
登录虚拟主机
```shell
docker-machine ssh master
docker-machine ssh slave
```
获得ip地址：
```
master：192.168.99.101
slave：192.168.99.102
```
## 二、在虚拟主机配置一主一从一哨兵
### 主redis
配置文件：redis_master.conf
```
daemonize no
pidfile "/var/run/redis.pid"
port 6379                       
timeout 300                     
loglevel warning                        
logfile "redis.log"                    
databases 1                        
rdbcompression yes                     
dbfilename "redis.rdb"                     
dir "/data"                    
requirepass password
masterauth password
maxclients 10000
maxmemory 1000mb                        
maxmemory-policy allkeys-lru                        
appendonly no                       
appendfsync alway
```
docker启动命令
```
docker run --name redis_master -p 6379:6379 -v $(pwd)/redis_master.conf:/data/redis_master.conf --restart=always -d redis:latest redis-server redis_master.conf
```
### 从redis
配置文件：redis_slave.conf
```
daemonize no
pidfile "/var/run/redis.pid"                       
port 6379                       
timeout 300                     
loglevel warning                        
logfile "redis.log"                    
databases 1                       
rdbcompression yes                     
dbfilename "redis.rdb"                     
dir "/data"                    
requirepass password
masterauth password
maxclients 10000                        
maxmemory 1000mb                        
maxmemory-policy allkeys-lru                        
appendonly no                       
appendfsync always                      
slaveof 192.168.99.101 6379
```
docker启动命令

```
docker run --name redis_slave -p 6379:6379 -v $(pwd)/redis_slave.conf:/data/redis_slave.conf --restart=always -d redis:latest redis-server 
```
### 哨兵
配置文件：sentinel.conf
```
daemonize no
port 26379
dir "/tmp"
sentinel monitor mymaster 192.168.99.101 6379 1
sentinel down-after-milliseconds mymaster 60000
sentinel auth-pass mymaster password
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
```
docker启动命令

```shell
docker run --name sentinel -p 26379:26379 -v $(pwd)/sentinel.conf:/data/sentinel.conf --restart=always -d redis:latest redis-sentinel sentinel.conf
```
## 三、测试主从同步、读写分离和哨兵监控
### 主动同步、读写分离
进入主redis
```
docker@master:~$ docker exec -it redis_master bash
root@4a788cdd0153:/data# redis-cli \\进入redis终端
127.0.0.1:6379> AUTH password \\登录
OK
127.0.0.1:6379> info replication \\查看信息
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.99.102,port=6379,state=online,offset=2679,lag=0
master_replid:24e1a9441d5df76943a737f890a293e054660653
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:2679
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:2679
127.0.0.1:6379> set cyf 123 \\设置键值对
OK
127.0.0.1:6379> keys * \\查看所有数据
1) "cyf"
127.0.0.1:6379> exit
root@4a788cdd0153:/data# exit
```
查看从redis

```
docker@slave:~$ docker exec  -it redis_slave bash
root@69d8af53d06f:/data# redis-cli 
127.0.0.1:6379> AUTH password
OK
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.99.101
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:131220
master_link_down_since_seconds:4
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:24e1a9441d5df76943a737f890a293e054660653
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:131220
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:131220 
```
查看数据是否同步
```
127.0.0.1:6379> keys *
1) "cyf" \\数据已同步
```
查看是否能写入数据
```
127.0.0.1:6379> set kb 123
(error) READONLY You can't write against a read only replica. \\从redis不可写入数据
127.0.0.1:6379> exit
root@4a788cdd0153:/data# exit
```
