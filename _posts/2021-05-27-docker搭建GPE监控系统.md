---
layout:     post
title:      docker搭建GPE监控系统
date:       2019-05-27
author:     cyf
header-img: img/cyf.png
catalog: true
tags:
    - GPE
    - Docker
---
[TOC]
# 一、prometheus
## 1、prometheus版本和镜像

|版本|镜像|
|:----:|:----:|
|2.7.1|prom/prometheus|

## 2、prometheus启动命令
```
docker run -d -p 9090:9090 -v /qj/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /qj/prometheus/rules/:/etc/prometheus/rules -v /qj/prometheus/data/:/prometheus -v /qj/prometheus/conf.d/:/etc/prometheus/conf.d --name prometheus --restart=always prom/prometheus --config.file=/etc/prometheus/prometheus.yml
```
## 3、prometheus配置文件

```
global: //全局设置，可以被覆盖
  scrape_interval: 15s //抓取目标实例的频率时间值，默认10s
  evaluation_interval: 15s //执行配置文件规则的频率时间值, 默认1m
  external_labels: //所有时间序列和警告与外部通信时用的外部标签
    monitor: 'codelab-monitor'

//Alertmanager配置
alerting:
 alertmanagers:
 - static_configs:
   - targets: 
      - 172.17.3.102:9093 //设定alertmanager和prometheus交互的接口，即alertmanager监听的ip地址和端口
    
rule_files: //规则文件指定规则文件路径列表。规则和警报是从所有匹配的文件中读取的
  - "/etc/prometheus/rules/*.yml" //自定义报警规则路径

scrape_configs: //抓取配置的列表
  - job_name: prometheus
    scrape_interval: 5s //抓取周期，默认采用global配置
    static_configs: //静态配置 
      - targets: ['172.18.0.1:9090'] //prometheus所要抓取数据的地址，即instance实例项
        labels:
          instance: prometheus
          
//一系列job_name
```
## 4、prometheus报警规则配置示例
[一些exporter镜像和报警规则](https://awesome-prometheus-alerts.grep.to/rules.html)
```
groups:
- name: 实例状态 
  rules:
  - alert: 实例状态down //告警名称
    expr: up == 0 //告警的判定条件，参考Prometheus高级查询来设定
    for: 1m  //满足告警条件持续时间多久后，才会发送告
    labels: //标签项
      team: node 
    annotations: //解析项，详细解释告警信息
      summary: '实例{{ $labels.instance }} 挂掉' //摘要
      description: '{{ $labels.instance }} 已经挂掉超过一分钟了.' //现状描述
```
### 4.1 告警信息生命周期的3中状态
> * inactive：表示当前报警信息即不是firing状态也不是pending状态
> * pending：表示在设置的阈值时间范围内被激活的
> * firing：表示超过设置的阈值时间被激活的

## 5、prometheus功能
> * 多维数据模型（时序由 metric 名字和 k/v 的 labels 构成）。
> * 灵活的查询语句（PromQL）。
> * 无依赖存储，支持local和remote不同模型。
> * 采用http协议，使用pull模式，拉取数据，简单易懂。
> * 监控目标，可以采用服务发现或静态配置的方式。
> * 支持多种统计数据模型，图形化友好。

## 6、时序类型
Prometheus 时序数据分为 Counter, Gauge, Histogram, Summary 四种类型
> * Counter 表示收集的数据是按照某个趋势（增加／减少）一直变化的，我们往往用它记录服务请求总量，错误总数等
> * Gauge 表示搜集的数据是一个瞬时的，与时间没有关系，可以任意变高变低，往往可以用来记录内存使用率、磁盘使用率等。
> * Histogram 由 <basename>_bucket{le="<upper inclusive bound>"}，<basename>_bucket{le="+Inf"}, <basename>_sum，<basename>_count 组成，主要用于表示一段时间范围内对数据进行采样，（通常是请求持续时间或响应大小），并能够对其指定区间以及总数进行统计，通常我们用它计算分位数的直方图
> * Summary 和 Histogram 类似，由 <basename>{quantile="<φ>"}，<basename>_sum，<basename>_count 组成，主要用于表示一段时间内数据采样结果，（通常是请求持续时间或响应大小），它直接存储了 quantile 数据，而不是根据统计区间计算出来的。

## 7、promql语法
[查询语法](https://www.yangcs.net/prometheus/4-prometheus/basics.html)
### 7.1 表达式语言数据类型
> * **瞬时向量**（Instant vector） - 一组时间序列，每个时间序列包含单个样本，它们共享相同的时间戳。也就是说，表达式的返回值中只会包含该时间序列中的最新的一个样本值。而相应的这样的表达式称之为瞬时向量表达式。
> * **区间向量**（Range vector） - 一组时间序列，每个时间序列包含一段时间范围内的样本数据。
> * **标量**（Scalar） - 一个浮点型的数据值。
> * **字符串**（String） - 一个简单的字符串值。

### 7.2 瞬时向量过滤器
瞬时向量过滤器允许在指定的时间戳内选择一组时间序列和每个时间序列的单个样本值，在{}里附加一组标签来进一步过滤时间序列。PromQL还支持用户根据时间序列的标签匹配模式来对时间序列进行过滤，目前主要支持两种匹配模式：完全匹配和正则匹配。总共有以下几种标签匹配运算符：
> * = : 选择与提供的字符串完全相同的标签。
> * != : 选择与提供的字符串不相同的标签。
> * =~ : 选择正则表达式与提供的字符串（或子字符串）相匹配的标签。
> * !~ : 选择正则表达式与提供的字符串（或子字符串）不匹配的标签。

**注意**：所有的 PromQL 表达式必须至少包含一个指标名称，或者一个不会匹配到空字符串的标签过滤器。

以下表达式是非法的（因为会匹配到空字符串）
```
{job=~".*"} # 非法！
```
以下表达式是合法的：
```
{job=~".+"}              # 合法！
{job=~".*",method="get"} # 合法！
```
### 7.3 区间向量过滤器
区间向量与瞬时向量的工作方式类似，唯一的差异在于在区间向量表达式中我们需要定义时间选择的范围，时间范围通过时间范围选择器[]进行定义，以指定应为每个返回的区间向量样本值中提取多长的时间范围。时间范围通过数字来表示，单位可以使用以下其中之一的时间单位：
> * s - 秒
> * m - 分钟
> * h - 小时
> * d - 天
> * w - 周
> * y - 年

### 7.4 时间位移操作
如果我们想查询，5 分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为 offset。

以下表达式返回相对于当前查询时间过去 5 分钟的 http_requests_total 值：
```
http_requests_total offset 5m
```
**注意**：offset 关键字需要紧跟在选择器 {}  后面。以下表达式是正确的：
```
sum(http_requests_total{method="GET"} offset 5m) #合法的
```
```
sum(http_requests_total{method="GET"}) offset 5m  #不合法的
```
### 7.5 操作符
#### 7.5.1 二元运算符
##### 算数二元运算符

|算数运算符|含义|
|:----:|:----:|
|+|加法|
|-|减法|
|*|乘法|
|/|除法|
|%|模|
|^|幂|

二元运算操作符支持 scalar/scalar(标量/标量)、vector/scalar(向量/标量)、和 vector/vector(向量/向量) 之间的操作。 
在两个标量之间进行数学运算，得到的结果也是标量。

在向量和标量之间，这个运算符会作用于这个向量的每个样本值上。例如：如果一个时间序列瞬时向量除以 2，操作结果也是一个新的瞬时向量，且度量指标名称不变,它是原度量指标瞬时向量的每个样本值除以 2。

如果是瞬时向量与瞬时向量之间进行数学运算时，过程会相对复杂一点，运算符会依次找到与左边向量元素匹配（标签完全一致）的右边向量元素进行运算，如果没找到匹配元素，则直接丢弃。同时新的时间序列将不会包含指标名称。
```
node_disk_bytes_written + node_disk_bytes_read //获取主机磁盘IO的总量
```
##### 布尔运算符

更多的用于设置报警规则

|布尔运算符|含义|
|:----:|:----:|
|==|相等|
|!=|不相等|
|>|大于|
|<|小于|
|\>=|大于等于|
|<=|小于等于|

##### 集合运算符

使用瞬时向量表达式能够获取到一个包含多个时间序列的集合，我们称为瞬时向量。 通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。 

|集合运算符|含义|
|:----:|:----:|
|and|并且|
|or|或者|
|unless|排除|

#### 7.5.2 匹配模式
##### 一对一匹配

一对一匹配模式会从操作符两边表达式获取的瞬时向量依次比较并找到唯一匹配(标签完全一致)的样本值。默认情况下，使用表达式：
```
vector1 <operator> vector2
```
在操作符两边表达式标签不一致的情况下，可以使用 on(label list) 或者 ignoring(label list）来修改便签的匹配行为。使用 ignoreing 可以在匹配时忽略某些便签。而 on 则用于将匹配行为限定在某些便签之内。

#### 7.5.3 聚合操作

Prometheus 还提供了下列内置的聚合操作符，这些操作符作用域瞬时向量。可以将瞬时表达式返回的样本数据进行聚合，形成一个具有较少样本值的新的时间序列。

|聚合操作符|含义|
|:----:|:----:|
|sum|求和|
|min|最小值|
|max|最大值|
|avg|平均值|
|stddev|标准差|
|stdvar|标准差异|
|count|计数|
|count_values|对 value 进行计数|
|bottomk|样本值最小的 k 个元素|
|topk|样本值最大的k个元素|
|quantile|分布统计|

这些操作符被用于聚合所有标签维度，或者通过 without 或者 by 子语句来保留不同的维度
```
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```
其中只有 count_values, quantile, topk, bottomk 支持参数(parameter)。

without 用于从计算结果中移除列举的标签，而保留其它标签。by 则正好相反，结果向量中只保留列出的标签，其余标签则移除。通过 without 和 by 可以按照样本的问题对数据进行聚合。

例如：

如果指标 http_requests_total 的时间序列的标签集为 application, instance, 和 group，我们可以通过以下方式计算所有 instance 中每个 application 和 group 的请求总量：
```
sum(http_requests_total) without (instance)
```
等价于
```
 sum(http_requests_total) by (application, group)
```
如果只需要计算整个应用的 HTTP 请求总量，可以直接使用表达式：
```
sum(http_requests_total)
```
count_values 用于时间序列中每一个样本值出现的次数。count_values 会为每一个唯一的样本值输出一个时间序列，并且每一个时间序列包含一个额外的标签。这个标签的名字由聚合参数指定，同时这个标签值是唯一的样本值。

例如要计算运行每个构建版本的二进制文件的数量：
```
count_values("version", build_version)
```
返回结果如下：
```
{count="641"}   1
{count="3226"}  2
{count="644"}   4
```
topk 和 bottomk 则用于对样本值进行排序，返回当前样本值前 n 位，或者后 n 位的时间序列。

获取 HTTP 请求数前 5 位的时序样本数据，可以使用表达式：
```
topk(5, http_requests_total)
```
quantile 用于计算当前样本数据值的分布情况 quantile(φ, express) ，其中 0 ≤ φ ≤ 1。

例如，当 φ 为 0.5 时，即表示找到当前样本数据中的中位数：
```
quantile(0.5, http_requests_total)
```
返回结果如下：
```
{}   656
```
#### 7.5.4 二元运算符优先级

在 Prometheus 系统中，二元运算符优先级从高到低的顺序为：
> * ^
> * *, /, %
> * +, -
> * ==, !=, <=, <, >=, >
> * and, unless
> * or

具有相同优先级的运算符是满足结合律的（左结合）。例如，2 * 3 % 2 等价于 (2 * 3) % 2。运算符 ^ 例外，^ 满足的是右结合，例如，2 ^ 3 ^ 2 等价于 2 ^ (3 ^ 2)。
### 7.6 内置函数

|函数|作用|
|:----:|:----:|
|rate()|计算范围向量中时间序列的每秒平均增长率|
|irate()|计算范围向量中时间序列的每秒即时增长率|
|topk()|按样本值计算的最大k个元素|
|absent()|如果传递给它的向量具有任何元素，则返回空向量;如果传递给它的向量没有元素，则返回值为1的1元素向量。|
|sum()|求和函数|

使用举例：
```
容器存活状态
rate(container_last_seen{instance=~"$container",name=~"$container_name"}[1m]) //[1m]代表过去一分钟

磁盘吞吐量，读的加上写的
irate(node_disk_read_bytes_total{instance=~"$host"}[1m]) + irate(node_disk_written_bytes_total{instance=~"$host"}[1m])

罗列出容器内存使用top10
topk(10,sum(container_memory_rss{instance=~"$container",name=~"$container_name"}) by (name))

判断容器是否存活，无值则代表容器存在，值为1则代表容器死亡
absent(container_last_seen{instance=~"$container",name=~"allqj"})
```
# 二、grafana
## 1、grafana版本和镜像

|版本|镜像|
|:----:|:----:|
|6.0.0|grafana/grafana|

## 2、grafana启动命令
```
docker run -d -p 3000:3000 --name grafana --restart=always -v /qj/grafana/data/:/var/lib/grafana -v /qj/grafana/logs:/var/log/grafana grafana/grafana
```
## 3、仪表盘规划
### 3.1 加载数据源
官方支持以下数据源：Graphite，InfluxDB，OpenTSDB，[Prometheus](http://docs.grafana.org/features/datasources/prometheus/)，Elasticsearch，CloudWatch。
### 3.2 面板使用
[Graph面板](http://docs.grafana.org/features/panels/graph/)

[Singlestat面板](http://docs.grafana.org/features/panels/singlestat/)

[Dashlist面板](http://docs.grafana.org/features/panels/dashlist/)

[Table面板](http://docs.grafana.org/reference/table_panel/)
# 三、alertmanager
## 1、alertmanager版本和镜像

|版本|镜像|
|:----:|:----:|
|0.16.1|prom/alertmanager|

## 2、alertmanager启动命令
```
docker run -d -p 9093:9093 -v /qj/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml --name alertmanager --restart=always prom/alertmanager --config.file=/etc/alertmanager/alertmanager.yml
```
## 3、alertmanager配置文件
```
global:
  resolve_timeout: 5m
route: //路由块定义路由树中的节点及其子节点
  receiver: webhook
  group_wait: 10s //当传入警报创建新的警报组时，请等待至少10s发送初始通知。
  group_interval: 1m //发送第一个通知时，请等待一分钟秒以发送一批为该组启动的新警报。
  repeat_interval: 1h //如果已成功发送警报，请等待一小时以重新发送它们。
  group_by: ['alertname'] //定义的分组

receivers:
- name: webhook
  webhook_configs:
  - url: http://172.17.3.102:8060/dingtalk/webhook/send 
    send_resolved: true

inhibit_rules: //一个inhibition规则是在与另一组匹配器匹配的警报存在的条件下，使匹配一组匹配器的警报失效的规则。两个警报必须具有一组相同的标签。
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
## 4、alertmanager功能
```
Alertmanager处理客户端应用程序（如Prometheus服务器）发送的警报。它负责对它们进行重复数据删除，分组和路由，以及正确的接收器集成。它还负责警报的静音和抑制。
```
# 四、pushgateway
## 1、pushgateway版本和镜像

|版本|镜像|
|:----:|:----:|
|0.7.0|prom/pushgateway|

## 2、pushgateway启动命令
```
docker run -d -p 9091:9091 --name pushgateway --restart=always prom/pushgateway
```
# 五、dingtalk_hook
## 1、dingtalk版本和镜像

|版本|镜像|
|:----:|:----:|
|无|registry.cn-beijing.aliyuncs.com/qianjia2018/qianjia_public:dingtalk-hook|

## 2、dingtalk启动命令
```
docker run -d -p 8060:8060 --name dingtalk-hook --restart=always registry.cn-beijing.aliyuncs.com/qianjia2018/qianjia_public:dingtalk-hook --ding.profile="webhook=https://oapi.dingtalk.com/robot/send?access_token=1bd8b136d437ddd7b93f2507def0576e0f77921dad3e3acbd758457246a41dff"
```
# 六、cadvisor
## 1、cadvisor版本和镜像

|版本|镜像|
|:----:|:----:|
|v0.32.0|google/cadvisor|

## 2、cadvisor启动命令
```
docker run -d -p 9000:8080 -v /:/rootfs:ro -v /var/run:/var/run:rw -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro -v /dev/disk/:/dev/disk:ro --name cadvisor --restart=always google/cadvisor
```
# 七、exporter
## 1、常用的exporter

范围|常用的Exporter
----|----
数据库|**MySQL Exporter**, **Redis Exporter**, MongoDB Exporter, MSSQL Exporter等
硬件|Apcupsd Exporter，IoT Edison Exporter， IPMI Exporter, **Node Exporter**等
消息队列|Beanstalkd Exporter, Kafka Exporter, NSQ Exporter, **RabbitMQ Exporter**等
存储|Ceph Exporter, Gluster Exporter, HDFS Exporter, ScaleIO Exporter等
HTTP服务|Apache Exporter, HAProxy Exporter, **Nginx Exporter**等
API服务|AWS ECS Exporter， Docker Cloud Exporter, Docker Hub Exporter, GitHub Exporter等
日志|Fluentd Exporter, Grok Exporter等
监控系统|Collectd Exporter, Graphite Exporter, InfluxDB Exporter, Nagios Exporter, SNMP Exporter等
其它|Blockbox Exporter, JIRA Exporter, Jenkins Exporter， Confluence Exporter等

## 2、node_exporter
### 2.1 版本和镜像 

|版本|镜像|
|:----:|:----:|
|0.17.0|prom/node-exporter|

### 2.2 启动命令
```
docker run -d -p 9100:9100 -v /proc:/host/proc:ro -v /sys:/host/sys:ro -v /:/rootfs:ro --net=host --name node-exporter --restart=always prom/node-exporter
```
## 3、mysql_exporter
[github地址](https://github.com/prometheus/mysqld_exporter)
### 3.1 版本和镜像

|版本|镜像|
|:----:|:----:|
|0.11.0|prom/mysqld-exporter|

### 3.2 启动命令
```
docker run -d -p 9104:9104 --name mysql_master --restart=always -e DATA_SOURCE_NAME="dbadmin:om123456@@(172.17.3.96:3316)/" prom/mysqld-exporter

docker run -d -p 9105:9104 --name mysql_slave1 --restart=always -e DATA_SOURCE_NAME="dbadmin:om123456@@(172.17.3.99:3316)/" prom/mysqld-exporter

docker run -d -p 9106:9104 --name mysql_slave2 --restart=always -e DATA_SOURCE_NAME="dbadmin:om123456@@(172.17.3.100:3316)/" prom/mysqld-exporter
```
## 4、rabbitmq_exporter
[github地址](https://github.com/kbudde/rabbitmq_exporter)
### 4.1 版本和镜像

|版本|镜像|
|:----:|:----:|
|无|kbudde/rabbitmq-exporter|

### 4.2 启动命令
```
docker run -d -p 9095:9090 -e RABBIT_URL="http://rabbitmq.allhome.com.cn" -e RABBIT_USER=admin -e RABBIT_PASSWORD=AllqjInter123@ --name rabbitmq-exporter --restart=always kbudde/rabbitmq-exporter
```
## 5、redis_exporter
github地址](https://github.com/oliver006/redis_exporter)
### 5.1 版本和镜像

|版本|镜像|
|:----:|:----:|
|无|oliver006/redis_exporter

### 5.2 启动命令
```
docker run -d -p 9121:9121 -e REDIS_ADDR=172.17.3.96:6380 -e REDIS_PASSWORD=qj12345678@ --name redis-exporter --restart=always oliver006/redis_exporter
```
## 6、nginx_exporter
## 7、process-exporter
### 7.1 启动命令
```
docker run -d -p 9256:9256 --privileged -v /proc:/host/proc:ro -v /qj/process/config/:/config --name process-exporter ncabatoff/process-exporter --procfs /host/proc -config.path /config/process.yml
```
### 7.2 process.yml
```
process_names:
  - cmdline:
      - '.+'
  - exe:
      - '.+'
  - comm:
      - '.+'
```