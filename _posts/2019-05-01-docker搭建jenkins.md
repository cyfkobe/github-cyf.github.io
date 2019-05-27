---
layout:     post
title:      docker搭建jenkins
date:       2019-05-01
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Docker
    - Jenkins
---
# docker启动命令
```
docker run -d -p 8080:8080 -p 50000:50000 -v /home/cyf/docker/jenkins/data/:/var/jenkins_home -u root --name jenkins jenkins/jenkins:lts
```
