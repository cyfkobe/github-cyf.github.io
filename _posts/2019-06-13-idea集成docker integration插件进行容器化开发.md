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
