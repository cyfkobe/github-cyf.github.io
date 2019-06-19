---
layout:     post
title:      idea集成docker integration插件进行容器化开发
date:       2019-06-13
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Idea
    - Docker
---
# 一、安装docker integration插件
打开idea`File-->Settings-->Plugins`，搜索`docker`然后安装即可

![搜索docker插件](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_search_docker.png?raw=true)
![安装docker插件](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_install_docker.png?raw=true)

可以看到底部多了一个docker图标,至此插件安装完毕

![底部标签](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_label_docker.png?raw=true)

# 二、使用docker integration插件
## 1. 设置镜像仓库地址

打开idea`File-->Settings-->Duild,Execution,Deployment-->Docker-->Registry`，填写好自己的镜像仓库地址，如下图所示

![设置镜像仓库地址](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_registry.png?raw=true)

## 2. 设置docker daemon

点击底部Docker按钮,左侧会有如下几个按钮

![点击docker按钮](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_button.png?raw=true)

点击`Edit Configuration`,一共有三种方式，如下图所示

![设置docker daemon](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_daemon.png?raw=true)

使用`TCP socket`需要开启指定宿主机的`Engine API`，修改`docker.service`，添加`-H tcp://0.0.0.0:2375`，重启docker即可（我的是Ubuntu系统）
```
cyf@KobeBryant:~$ cat /etc/systemd/system/multi-user.target.wants/docker.service
。。。
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock --containerd=/run/containerd/containerd.sock
。。。
cyf@KobeBryant:~$ sudo systemctl daemon-reload
cyf@KobeBryant:~$ sudo systemctl restart docker
```
## 3. 设置构建规则

点击`Deploy`按钮，会有如下图所示的三种构建方式

![点击部署按钮](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_deploy3.png?raw=true)

选择`Create Dockerfile Deployment...`各个参数按需设置好即可，设置示例如下

![选择dockerfile构建示例](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_deploy_dockerfile.png?raw=true)

然后点击run构建即可,构建结果如下图所示

![构架结果](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_deploy_result.png?raw=true)

同时可以对宿主机容器、镜像进行一系列操作，其它构建方式类似，不再做具体说明

# 三、idea开发流程
## 1. 连接docker主机API
打开`File-->Settings-->Duild,Execution,Deployment-->Docker`点击加号，添加docker主机连接API（配置好不同环境），如下图所示
![API添加](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_api.png?raw=true)
**注意**：需要docker主机开启API

## 2. 设置启动方式
打开`Run-->Edit Configurations`点击加号，如下图所示
![run设置](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_run.png?raw=true)
设置启动方式名称，不同环境对应不同的docker主机，并且设置好容器的`-d、-p、-v、-e`等等参数（具体按实际需求来），如下图所示
![run设置dev](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_run_dev.png?raw=true)
然后就会有如下启动方式
![启动类型](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_run_type.png?raw=true)

## 3. 添加Dockerfile文件
文件示例
```
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD target/*.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
## 4. 本地打包（配合Dockerfile）
```
mvn package -Dmaven.test.skip=true //是否跳过单元测试
```
## 5. 部署项目
点击刚才设置的启动方式或点击`Deploy`按钮，如下图所示即开始部署
![部署1](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_run_deploy1.png?raw=true)

![部署2](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/idea_docker_run_deploy2.png?raw=true)