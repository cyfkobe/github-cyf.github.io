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
## 2. 使用mysql数据库
创建数据库
```
shell> docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 1347445564/mysql5.7.18last
shell> mysql -h127.0.0.1 -uroot -p123456
mysql> create user sonar identified by 'sonar';
mysql> grant all privileges on *.* to sonar@'%' identified by 'sonar';
mysql> create database sonar default character set utf8 collate utf8_general_ci;
```
启动sonar
```
docker run -d -p 9000:9000 --name sonarqube --link mysql:mysql -e SONARQUBE_JDBC_URL="jdbc:mysql://mysql:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false" sonarqube:zh_CN
```
# 二、如何使用sonar
## 1. maven集成sonar
pom.yml添加sonar插件
```
    <build>
        <plugins>
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
执行`mvn clean install`,然后再执行`mvn sonar:sonar`,看到如下信息即可
```
[INFO] --- sonar-maven-plugin:3.6.0.1398:sonar (default-cli) @ authentication_java ---
[INFO] User cache: /home/cyf/.sonar/cache
[INFO] SonarQube version: 7.7.0
[INFO] Default locale: "zh_CN", source code encoding: "UTF-8"
[INFO] Load global settings
[INFO] Load global settings (done) | time=146ms
[INFO] Server id: 46AF5D23-AWtKmnxcvi-PoNSkn5al
[INFO] User cache: /home/cyf/.sonar/cache
[INFO] Load/download plugins
[INFO] Load plugins index
[INFO] Load plugins index (done) | time=64ms
[INFO] Plugin [l10nzh] defines 'l10nen' as base plugin. This metadata can be removed from manifest of l10n plugins since version 5.2.
[INFO] Load/download plugins (done) | time=141ms
```
**注意**:此时会有一个问题,代码覆盖率为`0.0%`,如下图
![sonar代码覆盖率为0](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/sonarcode.png?raw=true)
还需要添加一个单元测试覆盖率插件JaCoCo,再次执行即可
```
    <build>
        <plugins>
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <configuration>
                    <includes>com.*</includes>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```
![sonar代码覆盖率不为0](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/havesonarcode.png?raw=true)
## 2. npm集成sonar


