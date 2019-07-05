---
layout:     post
title:      docker搭建skywalking
date:       2019-07-05
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - 微服务
---

# 一、docker启动skywalking
```
shell> git clone https://github.com/JaredTan95/skywalking-docker.git
shell> cd skywalking-docker/6.x/docker-compose/
shell> docker-compose up -d
```
访问http://localhost:8080

# 二、java探针的使用
## 1. dockerfile中添加
```
FROM registry.cn-beijing.aliyuncs.com/cyf/all:oraclejdk-skywalking

VOLUME /tmp
COPY target/*.jar app.jar

ENTRYPOINT java -javaagent:/opt/skywalking/agent/skywalking-agent.jar=agent.service_name=service_name,collector.backend_service=127.0.0:11800 -Djava.security.egd=file:/dev/./urandom -jar app.jar
```