---
layout:     post
title:      kubernetes基础概念
date:       2019-06-01
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Kubernetes
---

- 自动装箱，自我修复，水平扩展，服务发现和负载均衡，自动发布和回滚
- 密钥和配置管理，存储编排，批量处理执行

集群
master/nodes

客户端-->master调度器-->nodes

master:API Server

master调度器：scheduler（评估系统资源，筛选最佳部署节点）

node上的kubelet，保证容器始终运行（副本）

master控制器（controller-manager）：监控容器状态（周期性探测容器是否正常运行）

master控制器管理器：监控控制器状态
    多个master做冗余，保证控制器管理器高可用
    
k8s直接调度的是pod，pod为k8s最小的调度逻辑单元，

pod内运行容器，pod内可以运行多个容器，多个容器共享同一个底层的网络名称空间（net uts ipc）
 
（user mnt pid）互相隔离


同一个pod内的容器共享存储卷，存储卷属于pod，各node主要用来运行pod

控制器管理pod

标识pod，打标签（key：value   app:nginx）,用于分类识别pod

标签选择器（selector）


# master/node
- master: API server,Scheduler(调度器),Controller-Manager(控制器)
- node: kubelet(集群代理-与API server交互),docker,...

# Pod, Label,Label Selector
- Label: key=value
- Label Selector: 

# Pod:（有生命周期）
1. 自主式Pod
2. 控制器管理的Pod(支持滚动回滚)
- ReplicationController
- ReplicaSet
- Deployment---HPA
- StatefulSet
- DaemonSet
- Job,Ctonjob
> HPA(自动伸缩器)

# service 
- client--->service（代理）--->pod
- 通过标签选择器关联pod

AddOns：附加组件

ipvs（net模型）：负载均衡

NTM

LBaaS

pod一个网络

service一个网络（集群网络）

节点一个网络

- 同一个pod内的多个容器：lo
- 各个pod之间的通信 overlay network(叠加网络)

