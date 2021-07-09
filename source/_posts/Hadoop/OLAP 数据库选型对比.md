---
type: blog
title: OLAP 数据库选型对比
date: 2021-06-17
categories: 大数据
tags: OLAP, 数据库
cover: false
meta:
  date: false
---





<!-- more -->

OLAP(OnLine Analysis Processing ，联机分析处理 ) 

直接给出结论：

- Hive、Hawq、Impala：基于 SQL on Hadoop
- Presto 和 Spark SQL 类似：基于内存解析 SQL 生成执行计划
- Kylin：用空间换时间、预计算
- Druid：数据实时摄入加实时计算
- ClickHouse：OLAP 领域的 HBase，单表查询性能优势巨大
- Greenpulm：OLAP 领域的 PostgreSQL

如果你的场景是基于 HDFS 的离线计算任务，那么 Hive、Hawq 和 Imapla 就是你的调研目标。

如果你的场景解决分布式查询问题，有一定的实时性要求，那么 Presto 和 SparkSQL 可能更符合你的期望。

如果你的汇总维度比较固定，实时性要求较高，可以通过用户配置的维度 + 指标进行预计算，那么不妨尝试 Kylin 和 Druid。

ClickHouse 则在单表查询性能上独领风骚，远超过其他的 OLAP 数据库。

Greenpulm 作为关系型数据库产品，性能可以随着集群的扩展线性增长，更加适合进行数据分析。



参考：

> [你需要的不是实时数仓 | 你需要的是一款强大的OLAP数据库(上)](https://mp.weixin.qq.com/s?src=11&timestamp=1623913265&ver=3135&signature=bAEiyYY0oioz9ZX5vVJW4lb0TXd5PkgKlQk7ODVzgRHSukwBgU8RFaelItgjETjUS0Hof7WebyikerbFatUTMP1UeKm5UrJPbkLtHWYhPPkHte1gwbsy5q4Ukm-zHoEW&new=1)
>
> [你需要的不是实时数仓 | 你需要的是一款强大的OLAP数据库(下)](https://mp.weixin.qq.com/s?src=11&timestamp=1623913265&ver=3135&signature=bAEiyYY0oioz9ZX5vVJW4lb0TXd5PkgKlQk7ODVzgRFui8AeZFx1Xxib4X55W7e-5y4CA9WBx6JzdDcAFPV8gZw*Vde4QLaUR1cbhTzMu83dt9W8Vblmt7xiZRkOXRz7&new=1)

