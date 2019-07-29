---
layout:     post
title:      docker搭建easy-mock
date:       2019-05-27
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Docker
---
# 一、首先启动mongo和redis
## 1.1 启动mongo

```
docker run -d -p 27017:27017 -v /qj/mongo/data/:/data/db --name mongo --restart=always mongo
```
进入容器：
```
docker exec -it mongo bash
```
运行：
```
mongo
use mock
```
## 1.2 启动redis

```
docker run -p 6379:6379 --name=redis -v /home/cyf/docker/redis/data:/data --restart=always -d  1347445564/redis-6379 redis-server --appendonly yes
```

# 二、启动easymock
在所要映射的目录下创建 default.json 和 local.json文件具体配置如下：
```
#default.json（默认配置文件）
{
    "port": 7300,
    "host": "0.0.0.0",
    "pageSize": 30,
    "proxy": false,
    "db": "mongodb://mongo:27017/easy-mock",
    "unsplashClientId": "",
    "redis": {
        "keyPrefix": "[Easy Mock]",
        "port": 6379,
        "host": "localhost",
        "password": "",
        "db": 0                          #redis所用库（可选）
    },
    "blackList": {
        "projects": [],
        "ips": []
    },
    "rateLimit": {
        "max": 1000,
        "duration": 1000
    },
    "jwt": {
        "expire": "14 days",
        "secret": "shared-secret"
    },
    "upload": {
        "types": [".jpg", ".jpeg", ".png", ".gif", ".json", ".yml", ".yaml"],
        "size": 5242880,
        "dir": "../public/upload",
        "expire": {
            "types": [".json", ".yml", ".yaml"],
            "day": -1
        }
    },
    "ldap": {
        "server": "",
        "bindDN": "",
        "password": "",
        "filter": {
            "base": "",
            "attributeName": ""
        }
    },
    "fe": {
        "copyright": "",
        "storageNamespace": "easy-mock_",
        "timeout": 25000,
        "publicPath": "/dist/"
    }
  }

```

```
#local.json
{
  "port": 7300,
  "host": "0.0.0.0",
  "pageSize": 30,
  "proxy": false,
  "db": "mongodb://172.22.0.1:27017/easy-mock",  #mongo的访问地址（docker网桥的IP地址加端口号，加库名）
  "unsplashClientId": "",
  "redis": {
    "port": 6379,
    "host": "172.22.0.1"                         #redis配置
  },
  "blackList": {
    "projects": [],
    "ips": [] 
  },
  "rateLimit": {
    "max": 1000,
    "duration": 1000
  },
  "jwt": {
    "expire": "14 days",
    "secret": "shared-secret"
  },
  "upload": {
    "types": [".jpg", ".jpeg", ".png", ".gif", ".json", ".yml", ".yaml"],
    "size": 5242880,
    "dir": "../public/upload",
    "expire": {
      "types": [".json", ".yml", ".yaml"],
      "day": -1
    }
  },
  "fe": {
    "copyright": "",
    "storageNamespace": "easy-mock_",
    "timeout": 25000,
    "publicPath": "/dist/"
  }
}

```
配置好配置文件后启动容器，连接mongo并做出映射：
```
docker run -d -p 7300:7300 --link mongo:mongo -v /qj/mock/config/:/easy-mock/config --name easymock blackcater/easy-mock
```
访问 http://localhost:7300 即可