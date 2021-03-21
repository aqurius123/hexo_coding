---
title: 会话技术
categories:
- 后端
tags: cookie\session
date:
---

> 使用场景：在现在得B/S结构系统中，使用会话技术记录用户信息的越来越多.

- 共性：cookie和session都是用来跟踪浏览器用户身份的会话方式；
- 区别：
1. cookie是存在于浏览器端的会话方式，而session则是存在于服务端的 
   会话方式；
2. cookie存储的数据量不大且不能保证数据安全，而session则相反；
   关联：session是基于cookie而存在的，也就是说必须有了cookie，才会用到session会话；