---
title: SpringCloud之Ribbon负载均衡实践篇
categories:
- 后端
tags: ribbon,feign
date:
---
> 紧接上篇，我们来总结下ribbon负载均衡相关知识点

### 回顾
回顾上篇文章，我们了解了Eureka注册中心集群的搭建及和ZK的类比，我们知道eureka的核心价值在于保证服务集群的高可用，那除了它内部的心跳检测及自我保护机制之外呢，ribbon提供来负载均衡的算法，将我们的请求均摊到集群的各个节点上，从而作为我们服务器降压及扩容的一个手段！

### 概念

