---
title: tomcat
categories:
- 其他
tags: tomcat
date: 2021-0-27
---

> 最近在启动tomcat时，本地项目始终报错：unrecognized windows sockets error:0 jvm_bind，经过分析属于8080端口被监听占用。

## 解决方案
1. netstat -ano | findstr 端口号：查看指定端口的使用情况；
2. netstat -ano | findstr 端口号：通过手动干掉端口占用进程；
