---
layout:     post
title:      docker搭建sonarqube代码审查
date:       2019-06-12
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Sonar
    - Docker
---

# 一、搭建sonar server
## 1. 使用postgresql数据库
搭建postgresql
```
docker run -d -p 5432:5432 --name postgresql -e POSTGRES_USER=sonar -e POSTGRES_PASSWORD=sonar postgres
```
重新构建sonar镜像,[下载汉化包](https://github.com/SonarQubeCommunity/sonar-l10n-zh/releases),Dockerfile如下(也可以添加其他插件):
```
FROM sonarqube
ADD sonar-l10n-zh-plugin-1.27.jar /opt/sonarqube/extensions/plugins/
```
```
docker build -t sonarqube:zh_CN .
docker run -d -p 9000:9000 --name sonar --link postgresql:postgresql -e SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql:5432/sonar sonarqube:zh_CN
```
默认账号密码:admin/admin,打开`http://localhost:9000`登录即可,登录界面如下
![登录界面](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/sonarlogin.png?raw=true)