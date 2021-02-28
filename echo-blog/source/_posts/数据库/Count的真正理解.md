---
title: Count
categories:
- 数据库
tags: Count
date: 2021-02-14
---

> count函数我们都知道是用来统计数据的，但是函数的真正含义你清楚吗？

## count的使用场景
1. count(*): 对表中的行数进行统计，不管表中列是NUll还是非空值，都会进行统计；

2. count(列名)：对表中指定列且该列为非空才会进行统计；
select count(1) from emp;----统计全部行数

select count(*) from emp;----统计全部行数

select count(COMM) from emp;----统计指定列且该列非空行数