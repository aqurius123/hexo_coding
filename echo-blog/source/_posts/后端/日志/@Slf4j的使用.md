---
title: 日志
categories:
- 后端/日志
tags: Slf4j
date:
---

> 使用场景：用于日志打印，只要引入相关依赖，就可以直接使用log.info()调用，方便在控制台直接打印出日志；

1、在service层和controller层都可以引入该注解，从而可以使用log对象调用日志查看，一般在service层使用居多；