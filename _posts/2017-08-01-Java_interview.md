---
layout: post
title: Interview
date: 2017-07-03 
tag: 学习总结
---

### 1.聚合索引与非聚合索引 ###

聚合索引:该索引中键值的逻辑顺序决定了表中数据的物理顺序.

	create clustered index 索引名 on 表(字段)


非聚合索引:数据存储在一个地方,索引存储在一个地方,索引带有指针指向数据的存储位置

	create nonclustered index 索引名 on 表(字段)
