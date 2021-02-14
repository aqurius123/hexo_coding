---
title: Oracle的初识
categories:
- 数据库
tags: Oracle
date:
---

## 常见数据库认识
1. Sql Servr: 微软公司开发的产品；
2. Db2: IBM公司开发;
3. Oracle: Oracle公司开发，中文名"甲骨文"公司；
> 引言：数据库的约束名称是可以任意取的，但是得有一定得命名规则,关键字constraint：
>> 1. 不能以数字开头；
>> 2. 一般按长约bai束简写+表级（列级）字段,例如：学生表的主键学号 PK_SNO、选课表的外键课号 FK_CNO;

> 约束的作用：
>> 1. 可以用作数据的校验不合法，例如CHECK约束可以对提交的数据进行限制，一定意义上起到录入校验的作用；

## Oracal常见字段类型，与Mysql略有不同：
1. Varchar2-------相当于Mysql的Varchar;
2. Number(length1, length2),length1表示整数的最大长度，length2表示小数点后的长度-----相当于Mysql的Double;
3. Clob: 存储大的文本，比如存储非结构化的XML文档
4. Blob: 二进制形式的长文本数据

