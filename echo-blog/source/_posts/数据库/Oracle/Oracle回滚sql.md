---
title: Oracle的回滚操作
categories:
- 数据库
tags: Oracle
date:
---

> Oracle的使用相对于Mysql、Db2等数据库略有不同，需要手动提交事务的情况比较多，这个对于Oracle的小白来说，刚开始是特别不习惯的，可能会出现一次性提交了多个事务而导致数据库中表的数据发生不是自己期望的变化，这个时候就需要我们对数据库的事务进行回滚的操作进行数据还原，具体操作如下(以cb_user_inf表为例)：

1、select * from cb_user_inf as of timestamp to timestamp('2020-07-21 09:30:00', 'yyyy-mm-dd hh24:mi:ss');

2、 alter table cb_user_inf enable row movement;

3、 flashback table cb_user_inf to timestamp to timestamp('2020-07-21 09:30:00', 'yyyy-mm-dd hh24:mi:ss');

