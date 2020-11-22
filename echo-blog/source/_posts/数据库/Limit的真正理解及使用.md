---
title: Limit&Order by的真正理解及使用
categories:
- 数据库
tags: Mysql
date:
---

## Limit的使用
> 作用：对查询的数据进行条数限制

> 场景1：只有一个参数，做限制条数
>> select DISTINCT  student_name, score, course_name from `score` limit 2----**注意：这是真正的简单限制条数**

> 场景2： 有2个参数，做限制条数
>> select DISTINCT  student_name, score, course_name from `score` limit 1,2;

>> **特别注意**：
>>> 1. 第一个参数是代表从第几行数据开始选择来做条数限制；
>>> 2. **这里0表示第一行数据，这也就是为什么有时分页需要根据前端传参的page-1的操作**

## Order by的使用
1. order by和数据库的默认排序方式一样，都是升序排序；
2. 当需要对多个字段进行排序时，且为降序排序时，一定要指定**降序排序**：
select student_name, score, course_name from `score` 
ORDER BY score desc, student_name;
> 注意：此时的排序方式是按照分数进行***降序***排序；如果分数相同，则会按照学生名字***升序***排序；如果分数不相同吗，则不会再去对student_name进行升序排序。





