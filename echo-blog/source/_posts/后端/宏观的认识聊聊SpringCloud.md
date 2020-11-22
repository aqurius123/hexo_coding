---
title: 宏观的认识聊聊SpringCloud
categories:
- 后端
tags: SpringCloud
date:
---
> 学了也用了这么久SpringBoot,你有没有思考过SpringBoot和SpringCloud 的关系呢？SpringCloud这么火的原因究竟有哪些呢？SpringCloud解决了哪些问题呢？Dubbo 和 SpringCloud 对比有什么不同呢？接下来我们就来带着问题，捋一捋。

### 宏观了解
MVC架构（三层架构）:结构清晰，方便开发人员协调分工，简化开发；

Spring(IOC、AOP): Spring是一个轻量级的JAVA框架，或者我们可以把它叫做一个IOC容器。它的诞生解决了企业级的开发复杂性问题，但是随着时间推移，越来越多的东西集成到Spring中，Spring也变得越来越复杂了；这似乎已经背离了Spring的初衷，再也不是简单易用的程序员的春天了！

那么SpringBoot应势头而生，它简化了Sprig的开发，我们可以理解为是Spring的升级版，是Spring的一个脚手架，提供了很多自动化配置的组件（约定大于配置)!于是乎，慢慢演进到现在成为了新一代的JavaEE的开发标准；

那么基于SpringBoot我们就可以快速开发一个企业级应用，企业级的单体应用因为SpringBoot的出现，又可以拆分成一个个的微服务应用。对比于以前的单体应用，业务逻辑没有发生任何改变，变的只是服务的架构，也叫微服务架构；

### 微服务架构
微服务架构解决的根本问题是解藕，本质就是模块化开发。那么这就带来了一个问题，服务多了后管理不是很方便啊，大大增加了运维及排错成本。相比单体的架构，它主要带来了以下四个问题：
- 这么多服务，客户端怎么访问服务端？
- 这么多服务，服务之间如何通信？
- 这么多服务，如何管理呢（又叫服务治理）？
- 这么多服务，服务挂了这么办？（运维成本大增）

那么SpringCloud也就这些问题应运而生，注意SpringCloud并不是一个技术，它是微服务架构的一站式解决方案，是基于SpringBoot的一个微服务生态圈！

那么解决上面问题，目前有哪些解决方案呢？
```
SpringCloud Netflix:
    api网关 -- zuul
    服务间通信 -- Feign (HttpClient-ribbon-feign)
    服务治理(服务注册与发现) -- Eureka
    服务熔断 -- Hystrix

Apache Dubbo + Zookeeper:
    api网关 -- 很遗憾，dubbo并没有开发自己的网关组件，目前使用了第三方提供的（例如:Zuul,Soul,Gateway...)
    服务间通信 -- 一个基于Java的Rpc通信框架
    服务治理(服务注册与发现) -- Zookeeper(多用于大数据相关，Hadoop、hive...)
    服务熔断 -- 也没有，用的是Hystrix

SpringCloud Alibaba:
    api网关 -- 也是用的zuul或Gateway
    服务间通信 -- 默认使用的也是Feign (HttpClient-ribbon-feign)
    服务治理(服务注册与发现) -- Nacos
    服务熔断 -- 也没有，默认使用的是Hystrix（可惜的是Hystrix也宣布不再维护了，官方推荐的替换版本是resilience4）
```
springCloud 官网：<a href="https://spring.io/projects/spring-cloud" target="_blank">https://spring.io/projects/spring-cloud</a>

SpringCloud 中文网：<a href="https://www.springcloud.cc/" target="_blank">https://www.springcloud.cc/</a>

Dubbo 官网：<a href="http://dubbo.apache.org/en-us/" target="_blank">http://dubbo.apache.org/en-us/</a>

Dubbo 中文手册：<a href="https://dubbo.gitbooks.io/dubbo-user-book/content/quick-start.html" target="_blank">https://dubbo.gitbooks.io/dubbo-user-book/content/quick-start.html</a>

SpringCloud Alibaba Wiki：<a href="https://github.com/alibaba/spring-cloud-alibaba/wiki" target="_blank">https://github.com/alibaba/spring-cloud-alibaba/wiki</a>

### 微服务概述
就目前而言，对于微服务，业界并没有一个统一的，标准的定义。
但通常而言，微服务架构是一种架构模式，或者说是一种架构风格，**它提倡将单一的应用程序划分成一组小的服务**，每个服务运行在其独立的自己的进程内，服务之间互相协调，互相配置，为用户提供最终价值。服务之间采用轻量级的通信机制互相沟通，每个服务都围绕着具体的业务进行构建，并且能够被独立的部署到生产环境中，另外，应尽量避免统一的，集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言，工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储；

**可能有的人觉得官方的话太过生涩，我们从技术维度来理解下**：

微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微服务提供单个业务功能的服务，一个服务做一件事情，从技术角度看就是一种小而独立的处理过程，类似进程的概念，能够自行单独启动或销毁，拥有自己独立的数据库。

关于微服务的详细理解，推荐博客: <a href="https://www.cnblogs.com/liuning8023/p/4493156.html" target="_blank">微服务（Microservices）——Martin Flower</a>

### 微服务优缺点
***优点***
- 每个服务足够内聚，足够小，代码容易理解，这样能聚焦一个指定的业务功能或业务需求；
- 开发简单，开发效率提高，一个服务可能就是专一的只干一件事；
- 微服务能够被小团队单独开发，这个小团队是2~5人的开发人员组成； 最少3个人！
- 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。
- 微服务能使用不同的语言开发。
- 易于和第三方集成，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如jenkins，Hudson，bamboo
- 微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过合作才能体现价值。
- 微服务允许你利用融合最新技术。
- 微服务只是业务逻辑的代码，不会和 HTML ， CSS 或其他界面混合
- 每个微服务都有自己的存储能力，可以有自己的数据库，也可以有统一数据库

***缺点***
- 开发人员要处理分布式系统的复杂性
- 多服务运维难度，随着服务的增加，运维的压力也在增大
- 系统部署间依赖增加
- 服务间通信成本增加
- 数据一致性问题
- 系统集成测试问题
- 性能监控问题

### 微服务目前技术栈
微服务条目 | 落地技术
-|-
服务开发 | SpringBoot,Spring,SpringMVC
服务配置与管理 | Netflix公司的Archaius、阿里的Diamond等
服务注册与发现 | Eureka、Consul、Zookeeper等
服务调用 | Rest、RPC、gRPC
服务熔断器 | Hystrix、Envoy等
负载均衡 | Ribbon、Nginx等
服务接口调用（客户端调用服务的简｜化工具） | Feign等
消息队列 | Kafka、RabbitMQ、ActiveMQ等｜
服务配置中心管理 | SpringCloudConfig、Chef等
服务路由（API网关） | Zuul等
服务监控 | Zabbix、Nagios、Metrics、Specatator等
服务部署 | Docker、OpenStack、Kubernetes等
数据流操作开发包 | SpringCloud Stream(封装与Redis，Rabbit，Kafka等发送接收消息)
事件消息总线 | SpringCloud Bus

### SpringCloud和Dubbo的对比
> 其实，它们注重的领域不一样，按理说没有太多的可比性.Dubbo的定位是一款RPC框架，Spring Cloud的目标是微服务架构下的一站式解决方案

-| Dubbo | Spring|
-| -| -
服务注册中心 | Zookeeper | Spring Cloud Netfilx Eureka
服务调用方式 | RPC | REST API
服务监控 | Dubbo-monitor|  Spring Boot Admin
断路器 | 不完善 | Spring Cloud Netflix Hystrix
服务网关 | 无 | Spring Cloud Netflix Zuul
分布式配置 | 无|  Spring Cloud Config
服务跟踪 | 无 | Spring Cloud Sleuth
消息总线 | 无|  Spring Cloud Bus
数据流 | 无 | Spring Cloud Stream
批量任务 | 无 | Spring Cloud Task

**最大区别**：SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。
严格来说，这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上
面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契
约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更加合适。

**品牌机与组装机的区别:**

很明显，Spring Cloud的功能比DUBBO更加强大，涵盖面更广，而且作为Spring的拳头项目，它也能够
与Spring Framework、Spring Boot、Spring Data、Spring Batch等其他Spring项目完美融合，这些对于微服务而言是至关重要的。

使用Dubbo构建的微服务架构就像组装电脑，各环节我们的选择自由度很高，但是最终结果很有可能因为一条内存质量不行就点不亮了，总是让人不怎么放心，但是如果你是一名高手，那这些都不是问题；

而Spring Cloud就像品牌机，在Spring Source的整合下，做了大量的兼容性测试，保证了机器拥有更高的稳定性，但是如果要在使用非原装组件外的东西，就需要对其基础有足够的了解。

**社区支持与更新力度**

最为重要的是，DUBBO停止了5年左右的更新，虽然2017.7重启了。对于技术发展的新需求，需要由开发者自行拓展升级（比如当当网弄出了DubboX），这对于很多想要采用微服务架构的中小软件组织，显然是不太合适的，中小公司没有这么强大的技术能力去修改Dubbo源码+周边的一整套解决方案，并不是每一个公司都有阿里的大牛+真实的线上生产环境测试过。

> 好，关于微服务的宏观理解及扩展就总结到这里；其实不要轻视这些思想层面的东西，因为这就是你的谈资；能做不代表会说，会说健谈也是一种能力的体现，做程序员这一行还是不能沉迷于闭门造车，交流总结很重要！后面就去开撸SpringCloud相关的东西了，奥利给！