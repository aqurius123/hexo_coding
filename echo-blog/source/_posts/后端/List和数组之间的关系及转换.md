---
title: List/Array
categories:
- 后端
tags: List
date:
---

关系：List的底层实现是动态数组；
数组的定义格式：
~~~
1. String[] array = new String[n]
2. String[] arr = new String{"aaa", "bbb"}
~~~
转换方式：
   - 数组转集合：Arrays.asList(array);
   - 集合转数组：list.toArray(array);------注意：此时数组一定要确定泛型，否则会抛出异常；


**特别注意**：Arrays.asList(array)方法虽然将数组转为了集合，但是仍然可以理解为特殊的集合(因为数组的长度是确定的)。
这个时候只能进行查和修改，不能进行增删操作，否则会抛出异常；