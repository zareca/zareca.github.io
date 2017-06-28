---
layout: post
title: 聚集索引和非聚集索引区别
date: 2017-06-28 
tag: 数据库相关
---

### 聚集索引 ###

该索引中键值的逻辑顺序决定了表中相应行的物理顺序

类似于字典按拼音查找

    create clustered index ix_TB on TBname(字段)

### 非聚集索引 ###

数据存储在一个地方,索引存储在一个地方,索引带有指针指向数据的存储位置.

类似于字典按偏旁查找

    create nonclustered index ix_TB on TBname(字段)

<img src="/images/posts/database/indexabout.png" height="280" width="600"> 