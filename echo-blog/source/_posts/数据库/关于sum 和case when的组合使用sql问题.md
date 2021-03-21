---
title: case when
categories:
- 数据库
tags: case when
date: 2021-02-16
---

SELECT SUM(case WHEN sex=1 then 1 else 0 end )as '男生',SUM(case when sex =2 then 1 else 0 end )'女生'FROM asex
通过case when 函数实现对多个信息项一次性查询出来进行展示；