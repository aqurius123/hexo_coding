---
title: 集合
categories:
- 后端
tags: List
date: 2021-04-18
---

## 集合常规操作
- add(增)
- remove(删)
- set(改)
- get(查)

## 集合使用注意事项
1. List是对象，其泛型也必须是对象，所以针对list的泛型必须指定为包装类，而不能使用基本数据类型,如：
> ArrayList<int> ints = new ArrayList<int>(); //编译错误
基本类型	引用类型
- boolean	Boolean
- byte	    Byte
- short	    Short
- int	    Integer
- long	    Long
- float	    Float
- double	Double
- char	    Character

## Collections工具类的使用
1. 排序---针对数字、字符等都可以进行排序；

## ArrayList常用Api
方法	描述
- add()	将元素插入到指定位置的 arraylist 中
- addAll()	添加集合中的所有元素到 arraylist 中
- clear()	删除 arraylist 中的所有元素
- clone()	复制一份 arraylist
- contains()	判断元素是否在 arraylist
- get()	通过索引值获取 arraylist 中的元素
- indexOf()	返回 arraylist 中元素的索引值
- removeAll()	删除存在于指定集合中的 arraylist 里的所有元素
- remove()	删除 arraylist 里的单个元素
- size()	返回 arraylist 里元素数量
- isEmpty()	判断 arraylist 是否为空
- subList()	截取部分 arraylist 的元素
- set()	替换 arraylist 中指定索引的元素
- sort()	对 arraylist 元素进行排序
- toArray()	将 arraylist 转换为数组
- toString()	将 arraylist 转换为字符串
- ensureCapacity()	设置指定容量大小的 arraylist
- lastIndexOf()	返回指定元素在 arraylist 中最后一次出现的位置
- retainAll()	保留 arraylist 中在指定集合中也存在的那些元素
- containsAll()	查看 arraylist 是否包含指定集合中的所有元素
- trimToSize()	将 arraylist 中的容量调整为数组中的元素个数
- removeRange()	删除 arraylist 中指定索引之间存在的元素
- replaceAll()	将给定的操作内容替换掉数组中每一个元素
- removeIf()	删除所有满足特定条件的 arraylist 元素
- forEach()	遍历 arraylist 中每一个元素并执行特定操作