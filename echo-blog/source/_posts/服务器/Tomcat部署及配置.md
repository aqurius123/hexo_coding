---
title: Tomcat
categories:
- 后端/服务器
tags: Tomcat
date:
---

> web项目依赖于tomcat容器进行部署；

## web项目的简单组成：
   1. pom文件；-------依赖的版本很是关键，涉及到版本依赖控制
   2. web.xml配置文件：
       a、初始化spring容器对象；
       b、配置前端控制器、监听器和过滤器等；
   3. springmvc.xml等配置文件：
       a、配置扫描包
       b、配置前缀、后缀
       c、开启注解扫描等
## tomcat的配置：
![image.png](1)
![image.png](2)-----配置本地tomcat的存放目录
![image.png](3)
![image.png](4)