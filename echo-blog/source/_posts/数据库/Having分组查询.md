---
title: Having分组查询
categories:
- 数据库
tags: having
date: 2021-02-14
---

> where是对行条件作出过滤筛选；而having是对分组进一步作出过滤筛选

--分组查询

select e.deptno, count(e.deptno) as counts from emp e 
group by e.deptno having count(e.deptno) >= 2;---正确

select e.deptno, count(*) as counts from emp e 
group by e.deptno having count(*) >= 2;---正确



select e.deptno, count(*) as counts from emp e 
group by e.deptno having counts >= 2;---语法错误

**注意**：
having后面不能使用别名来作为条件过滤，必须和select查询中保持一致，否则会语法报错；