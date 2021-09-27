---
type: blog
title: JAVA 编程小技巧
date: 2021-08-26
categories: 技术
tags: JAVA
cover: false
meta:
  date: false
---



模板～

<!-- more -->



### map 对不存在的 key 赋值

```java
map.put(key, map.getOrDefault(key, 0) + 1)
```





