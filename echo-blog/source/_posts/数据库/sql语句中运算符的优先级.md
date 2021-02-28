---
title: Sql语句中运算符的优先级
categories:
- 数据库
tags: 运算符
date:
---

## Sql的逻辑处理顺序：从高到低
from -> 
where -> 
group by -> 
having -> 
select(over -> distinct -> top) -> order by ->
limit(Mysql特有语法)

## 运算符的优先处理顺序：从高到低
() ->
*或/ -> 
+或- -> 
=或>或< -> 
not -> 
and -> 
between/In/like/or -> 




## 注意
1. 在某些sql语句中or和in的实现意义是一样的，但是in的可读性更高且执行更快；