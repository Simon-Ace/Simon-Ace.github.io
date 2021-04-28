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



**例子：**

gauge 类型数据，可以用`xxx_over_time()`函数来做一段时间的数据聚合操作

quantile_over_time(0.3, yarn_label_used_capacity[1d])



> [prometheus 函数语法介绍](https://prometheus.io/docs/prometheus/latest/querying/functions/)
>
> [一篇文章带你理解和使用Prometheus的指标](https://frezc.github.io/2019/08/03/prometheus-metrics/)



## 三、Grafana 展示 & 报警

