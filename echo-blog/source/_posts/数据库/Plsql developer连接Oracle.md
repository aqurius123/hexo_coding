---
title: Plsql developer连接Oracle
categories:
- 数据库
tags: Plsql developer
date:
---

> 作为数据库中的大佬Oracle，很多时候我们不得不对它进行关注，尽管Mysql数据库的应用比较广泛。今天，就聊一聊作为小白对对oracle的初步认识及使用，以及oracle的搭档————plsql developer的初步使用；Oracle的安装这里就不作叙述了，都可以进行百度，主要是讲一讲自己第一次使用Oracle碰到过的相关问题及解决方案；

## 安装Oracle可能会出现的坑:
有的安装文档可能没有提及到关于Oracle数据库的系统环境变量的配置，这样可能导致后面玩plsql developer的时候死活出现not logged on的错误提示~~~有点坑，还是得吃这个亏。。。。。
[详情参考安装文档](https://blog.csdn.net/zzybbh/article/details/83818689)

## Oracle所谓的一次性插入多条数据
首先，Oracle的主键是不能像mysql一样设置自增的，必须通过序列或者算法等其他手段来进行处理，所以在你执行一次性插入多条时，必须保证插入的每一列都是实实在在的参数，例如：

insert all 
into first_oracle (id, u_name,u_age) values (2, '张三', '20')
into first_oracle (id, u_name,u_age) values (3, '李四', '40')
select 1 from dual;----这是正确的

> ----
insert all 
       into first_oracle values(seq_first_oracle.nextval,'甲','2')
       into first_oracle values(seq_first_oracle.nextval,'乙','4')
       into first_oracle values(seq_first_oracle.nextval,'丙','1')
       into first_oracle values(seq_first_oracle.nextval,'丁','2')
       into first_oracle values(seq_first_oracle.nextval,'丙','5')
select 1 from dual;---这个是有问题的，因为seq_first_oracle作为序列，不能保证事务正常提交，导致无法正常执行；

