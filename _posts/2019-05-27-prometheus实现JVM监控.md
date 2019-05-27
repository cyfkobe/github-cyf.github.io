---
layout:     post
title:      prometheus实现JVM监控
date:       2019-05-27
author:     cyf
header-img: img/cyf.png
catalog: true
tags:
    - GPE
    - JVM
---
# java程序pom.xml添加如下依赖包
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
```
**注意**：如果出现datasource循环依赖添如下配置
```
spring:
  cloud:
    refresh:
      refreshable: none
```
# java程序application-dev.yml和application-prod.yml添加如下配置
```
management:
  endpoint:
    health:
      show-details: always
    metrics:
      enabled: true
    prometheus:
      enabled: true
  endpoints:
    web:
      exposure:
        include: "*"
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
```
**注意**：公共包要用1.0.8的版本，本地启动需要删除本地.m2目录下的该依赖包，重新加载该依赖包
```
        <dependency>
            <groupId>com.allqj.qjf</groupId>
            <artifactId>common</artifactId>
            <version>1.0.8-RELEASE</version>
        </dependency>
```
# 最终效果：访问 http://localhost:端口号/actuator/prometheus
```
# HELP tomcat_servlet_request_max_seconds  
# TYPE tomcat_servlet_request_max_seconds gauge
tomcat_servlet_request_max_seconds{application="checkin",name="default",} 0.0
tomcat_servlet_request_max_seconds{application="checkin",name="dispatcherServlet",} 0.0
# HELP hikaricp_connections_pending Pending threads
# TYPE hikaricp_connections_pending gauge
hikaricp_connections_pending{application="checkin",pool="HikariPool-1",} 0.0
# HELP jdbc_connections_max  
# TYPE jdbc_connections_max gauge
jdbc_connections_max{application="checkin",name="dataSource",} 10.0
# HELP tomcat_threads_busy_threads  
# TYPE tomcat_threads_busy_threads gauge
tomcat_threads_busy_threads{application="checkin",name="http-nio-8002",} 1.0
# HELP tomcat_sessions_active_current_sessions  
# TYPE tomcat_sessions_active_current_sessions gauge
tomcat_sessions_active_current_sessions{application="checkin",} 0.0
# HELP hikaricp_connections Total connections
# TYPE hikaricp_connections gauge
hikaricp_connections{application="checkin",pool="HikariPool-1",} 10.0
# HELP tomcat_global_request_max_seconds  
# TYPE tomcat_global_request_max_seconds gauge
tomcat_global_request_max_seconds{application="checkin",name="http-nio-8002",} 0.0
# HELP jvm_gc_pause_seconds Time spent in GC pause
# TYPE jvm_gc_pause_seconds summary
jvm_gc_pause_seconds_count{action="end of minor GC",application="checkin",cause="Allocation Failure",} 33.0
```
# prometheus配置prometheus.yml
## 第一种方式：手动添加
```
  - job_name: checkin #以application name区分
    scrape_interval: 5s
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['192.168.10.200:8002']
        labels:
          instance: checkin #以application name区分
```
## 第二种方式：eureka服务自动发现
```
  - job_name: eureka
    scheme: http
    metrics_path: '/actuator/prometheus'
    consul_sd_configs:
    #consul 地址
      - server: '192.168.10.200:1025' #eureka服务地址
        scheme: http 
        services: [CHECKIN] #eureka上注册的服务名，没有则代表所有服务
```
**注意第二种方式的实现需要eureka添加如下依赖包**
```
        <dependency>
            <groupId>at.twinformatics</groupId>
            <artifactId>eureka-consul-adapter</artifactId>
            <version>${eureka-consul-adapter.version}</version>
        </dependency>
        
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
**eureka服务相应版本要求**
- Java 1.8+

版本1.1.x及更高版本
- Spring Boot 2.1.x
- Spring Cloud Greenwith

版本1.0.x及更高版本
- Spring Boot 2.0.x
- Spring Cloud Finchley

版本0.x
- Spring Boot 1.5.x
- Spring Cloud Edgware

详细文档见：[SpringCloud使用Prometheus監控(基於Eureka)](https://www.jishuwen.com/d/2M4h/zh-tw)
