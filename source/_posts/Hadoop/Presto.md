---
type: blog
title: Presto
date: 2021-02-04
categories: 教程
tags: Presto, 教程
cover: false
meta:
  date: false
---

# Presto

## 一、 简介

### 1.1 概念

- 是什么：即席查询框架
- 为什么出现：
  - hive等大数据仓库查数据太慢
  - 原来的数据库（仓库）无法做到跨源联合查询
- presto特征：
  - 基于内存
    - 数据量支持 GB~PB 查询，查询在几秒到几分钟
  - 可联合多个数据源进行join查询
  - 只是一个查询引擎，不存储数据

### 1.2 架构

![image-20210204102243401](https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210204102243401.png)

### 1.3 优缺点

**优点：**

- 基于内存，减少落盘次数，计算快
- 能够连接多个数据源，进行跨数据源join

**缺点：**

- 因为是放在内存计算，当连表查询时，会产生一堆的临时表（落盘？），反而比hive还慢



