---
title: 反射
categories:
- 后端/反射
tags: 反射
date:
---    

> 反射的作用异常强大，在工作中经常会被用到，只要使用得当，会非常方便，
下面针对3种不同场景进行获取当前类的字节码class对象的获取：

1. 非静态方法中获取当前类的字节码class对象
   通过  ++this.getClass()++  方式来获取
2. 静态方法中获取字节码class对象
   通过++类名.class++直接获取
3. 任何地方获取字节码class对象
   通过Class.forName(String param)获取
   注意：param指的是当前类的全限定类名(包名+类名)；
![image.png](1)
