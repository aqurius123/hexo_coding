---
title: equals
categories:
- 后端
tags: equals
date:
---

> 都知道hashcode()和equals()方法都是比较两者是否相等的方法，但是何时使用hashcode()？又何时使用equals()方法呢？两者又有何区别呢？

- 共同点：两者都是用来比较两者是否相等；
- 不同点：
    1. hashcode()方法的内部实现比较简单，而equals()方法内部实现 更加全面，当然也更加复杂；
    2. hashcode()执行的速度较快，因为相对equals()方法内部实现相对简单；
    3. hashcode()用来获取当前对象的hash值，一般来说，hash值相同的两者，可能是同一个对象------注意：仅仅是可能; hash值不同的两者一定不相同；

**注意**：一般严格意义上讲，要是判断两者是否相同，先判断hash值是否相等，再通过equals()方法是非常准确的；

