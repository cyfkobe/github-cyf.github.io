---
layout:     post
title:      mysql开启gtid复制模式
date:       2019-05-06
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Mysql
---
# 一、配置
MySQL5.6
```
gtid_mode=ON(必选)    
log_bin=ON(必选)    
log-slave-updates=ON(必选)    
enforce-gtid-consistency=true(必选)
```
MySQL5.7.13 or higher
```
gtid_mode=ON(必选)  
enforce-gtid-consistency=true(必选)
log_bin=ON（可选）--高可用切换，最好设置ON  
log-slave-updates=ON（可选）--高可用切换，最好设置ON
**注意**：当开启GTID模式时，集群中的全部MySQL Server必须同时配置gtid_mod = ON，否则无法同步
```
**注意：当开启GTID模式时，集群中的全部MySQL Server必须同时配置gtid_mod = ON，否则无法同步**

```
mysql> show slave status\G
...
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 55e6ffed-2e94-11e9-8d91-0242ac120002:1-5 //执行的第1到第5个数据库事务
            Executed_Gtid_Set: 55e6ffed-2e94-11e9-8d91-0242ac120002:1-5
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

```
Retrieved_Gtid_Set：从库已经接收到主库的事务编号
Executed_Gtid_Set：已经执行的事务编号
```
# 二、gtid跳过复制错误的方法
```
stop slave;
set gtid_next='2a09ee6e-645d-11e7-a96c-000c2953a1cb:1-10';
begin;
commit;
set gtid_next='AUTOMATIC';
start slave; //只能跳过一个事务
```
```j
stop slave;
reset master;
set global gtid_purged='2a09ee6e-645d-11e7-a96c-000c2953a1cb:1-33';