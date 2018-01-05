---
layout: post
title: Mysql 学习笔记 —— 删除某个字段的重复数据
date: 2016-12-06 12:04:01
description: mysql 删除重复数据
img: mysql_delete_repetitive_data.png
tags: [mysql]
---

当我们要删除 xyj_works_text 这张表中 works_id 字段重复的数据，我们可以选择保留相同 works_id 中，id 较大的那条纪录，代码如下：

```sql
delete from xyj_works_text where id not in (
	select id from (
		select max(id) as id from xyj_works_text group by works_id
	) as b
)
```
	

