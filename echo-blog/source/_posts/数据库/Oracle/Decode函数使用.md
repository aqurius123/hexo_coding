---
title: decode
categories:
- Oracle
tags: decode
date:
---

> 语法
decode(条件，值1，返回值1，值2，返回值2，...值n,返回值n，缺省值) 

## 函数含义
IF条件=值1THEN

RETURN(返回值1)

ELSIF条件=值2THEN

RETURN(返回值2)

......

ELSIF条件=值nTHEN

RETURN(返回值n)

ELSE

RETURN(缺省值)

**注意：缺省值就是默认值，当条件不匹配时，指定查询结果为默认值（如：''）**