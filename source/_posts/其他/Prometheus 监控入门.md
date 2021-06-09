---
type: blog
title: Prometheus 监控入门
date: 2021-04-28
categories: 技术
tags: prometheus
cover: false
meta:
  date: false
---





<!-- more -->

## 一、服务暴露接口

需要暴露出 metrics 接口



## 二、Prometheus 对接

在 prometheus 中配置服务的 ip:port



## 三、PromQL

### 3.1 查询时间序列

**匹配模式**

- 完全匹配

通过使用`label=value`可以选择那些标签满足表达式定义的时间序列

反之使用`label!=value`则可以根据标签匹配排除时间序列

- 正则匹配

使用`label=~regx`表示选择那些标签符合正则表达式定义的时间序列

反之使用`label!~regx`进行排除

### 3.2 范围查询

时间范围通过时间范围选择器`[]`进行定义。例如，通过以下表达式可以选择最近5分钟内的所有样本数据：

```
http_requests_total{}[5m]
```

### 3.3 时间位移

想查询，5分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为**offset**。

可以使用offset时间位移操作：

```
http_request_total{} offset 5m
http_request_total{}[1d] offset 1d
```

### 3.4 聚合

对同一特征多个label的序列，进行聚合操作

```
# 查询系统所有http请求的总量
sum(http_request_total)

# 按照mode计算主机CPU的平均使用时间
avg(node_cpu) by (mode)

# 按照主机查询各个主机的CPU使用率
sum(sum(irate(node_cpu{mode!='idle'}[5m]))  / sum(irate(node_cpu[5m]))) by (instance)
```





**例子：**

gauge 类型数据，可以用`xxx_over_time()`函数来做一段时间的数据聚合操作

quantile_over_time(0.3, yarn_label_used_capacity[1d])



> [prometheus 函数语法介绍](https://prometheus.io/docs/prometheus/latest/querying/functions/)
>
> [一篇文章带你理解和使用Prometheus的指标](https://frezc.github.io/2019/08/03/prometheus-metrics/)
>
> [★Prometheus Book](https://yunlzheng.gitbook.io/prometheus-book/)
>
> 



## 三、Grafana 展示 & 报警

> [grafana配置仪表盘](https://blog.csdn.net/qingwufeiyangxz/article/details/108659681)
>
> [Prometheus运维七 详解可视化Grafana工具](https://blog.csdn.net/ZhanBiaoChina/article/details/107048324)