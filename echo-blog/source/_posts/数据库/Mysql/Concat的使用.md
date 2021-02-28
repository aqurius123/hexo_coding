---
title: Concat
categories:
- 数据库
tags: concat
date: 2021-02-14
---

> 在sql语句中，不仅仅是一个简单的字段查询，更多的会涉及到对一个字段乃至多个字段进行操作后展示；
concat作为一个*Mysql*中字符串拼接的语法经常会被用到；而*Oracle*中则会用到||来实现字符串的拼接-----需要注意；

## Concat的语法结构concat(arg1, arg2, ...)
select CONCAT(u.u_id,'对应的用户名称为：', u.u_name) from test_user u;

## Oralce中||的用法
select (e.empno || '编号对应的员工名字为：' || e.ename) A   from emp e;

## 个人发现
Oracle语法中是不能将中文字符串作为别名使用的，
但是Mysql中是被允许的：
select u.u_name as '员工姓名' from test_user u;---Mysql
select e.empno as '编号' from emp e;----Oracle中会报错

