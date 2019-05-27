---
layout:     post
title:      Ubuntu安装破解navicat
date:       2019-05-08
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Mysql
---
# 1、首先下载navicat
[官网下载地址](https://www.navicat.com.cn/download/navicat-for-mysql)
选择linux最新版本：我用的是navicat121_mysql_cs_x64.tar.gz，解压到指定目录
```
tar -zxvf /path/to/navicat121_mysql_cs_x64.tar.gz -C /opt
mv /opt/navicat121_mysql_cs_x64 navicat
```
# 2、解决乱码问题
编辑start_navicat
```
vim /opt/navicat/start_navicat

# Wine environment variables
WINEDIR="wine"
export LANG="zh_CN.utf8" //修改成中文
```
`./start_navicat`进入navicat页面，修改：`工具栏` --> `选项` 下的字体为`AR PL UMing CN`
![常规](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/%E5%B8%B8%E8%A7%84.png?raw=true)

![编辑器](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/%E7%BC%96%E8%BE%91%E5%99%A8.png?raw=true)

![记录](https://github.com/github-cyf/github-cyf.github.io/blob/master/img/%E8%AE%B0%E5%BD%95.png?raw=true)

# 添加快捷方式
在`/usr/share/applications`路径下添加`navicat.desktop`配置如下:
```
vim /usr/share/applications/navicat.desktop

[Desktop Entry]                                                                                      
Encoding=UTF-8
Name=Navicat
Comment=The Smarter Way to manage dadabase
Exec=/opt/navicat/start_navicat
Icon=/opt/navicat/navicat.png //自己去网上下载个图片即可
Categories=Application;Database;MySQL;navicat
Version=1.0
Type=Application
Terminal=0
```
# 到期之后的解决方案
删除`~/.navicat`文件
```
rm -rf ~/.navicat
```
重新启动后再做一下第2步的操作即可
[本文摘自Noobln江湖](https://blog.csdn.net/qq_41376740/article/details/80499545)