---
title: SpringCloud之Eureka
categories:
- 后端
tags: eureka
date:
---

> 上篇我们了解了SpringCloud的一个整体架构，我们知道Eureka是Netflix的一个核心组件，那么我们有必要彻底去了解一下它；废话不多说，我们开始

### 概念
Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务发现功能。

Eureka包含两个组件：**Eureka Server**和**Eureka Client**。

**Eureka Server**提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。

**Eureka Client**是一个java客户端，用于简化与Eureka Server的交互，客户端同时也就是一个内置的、使用轮询(round-robin)负载算法的负载均衡器。

注：Eureka 遵循AP原则
CAP 原则：c -> 一致性;A -> 可用性；p -> 分区容错性；
和Zookeeper 相比，zookeeper遵循CP原则；
具体可以参考博客：<a href="https://www.jianshu.com/p/5c5753d2aeb0" target="_blank">Eureka与Zookeeper对比</a>

### 快速开始
开始之前我们回顾下不用注册中心，我们都是怎么调用的
httpClient或者restTemplate，这里我们回顾下RestTemplate的 写法如下：
```java
@Configuration
public class HttpConfig {

    @Bean
    public RestTemplate setRestTemplate(){
        return new RestTemplate();
    }
}
@GetMapping("/call")
public String call(){
    return restTemplate.getForObject("localhost:xxxx/test/call",String.class);
}
```
那么这种写法，相当于写死了通信地址，硬编码格式；那么eureak 就是提供了一个地方动态的感知这些服务，让处于一个注册中心或者注册中心集群内的客户端间能互相通信，并且ribbon 和feign提供了负载均衡的算法，能让服务在多个节点上访问均匀分布以此来缓解单台服务的压力；

### 注册中心搭建
新建项目eureka-test,新建子模块eureka-server9001作为注册中心服务端

导入依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```
yaml配置
```yaml
server:
  port: 9001

# eureka 配置
eureka:
  instance:
    # 实例名称
    hostname: localhost
    #appname: eureka-server9001
  client:
    fetch-registry: false #当为单例时是否从eureka上获取服务的具体信息
    fetch-register-with-eureka: true #当为单例时是否将自己注册到eureka
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
spring:
  application:
    name: eureka-server-9001
```
启动测试
记得启动类上加上`@EnableEurekaServer`
![](https://s1.ax1x.com/2020/05/27/tFxRrq.png)

注⚠️：注册中心的实例名即配置的`appname` 如果未设置将默认为配置的`spring.application.name`
从截图我们看到，服务端9001自己注册上去了，我们只需要将配置文件中的`fetch-register-with-eureka`改为false即可，另外你也可以将`fetch-registry`改为true 看下结果哈，或者将两个配置都改为true，你将会有神奇的发现；

注⚠️：当仅有自己一个节点时，以上两个配置才会生效。当有多个实例子时，服务端都会将自己注册到注册中心；

### 创建eureka客户端实例

创建新模块，eureka-client8001,eureka-client8002；并将它们注册到9001的eureka 服务端上去，以实现客户端的通信

同上引入eureka的starter依赖，启动类上加上`@EnableEurekaClient`注解
***小技巧：***
为了后面可以看到实例信息，我们将8001依赖及配置改写如下：
```xml
<!--新增依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<!-- maven 构建-->
<build>
    <finalName>spring-eureka</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <delimiters>
                    <!--以$ 开头或结尾的在src/main/resources下的配置就可以被分割读取到-->
                    <delimiter>$</delimiter>
                </delimiters>
            </configuration>
        </plugin>
    </plugins>
</build>
```
yaml配置如下:
```yaml
server:
  port: 8001

# eureka 配置
eureka:
  instance:
    prefer-ip-address: true #注册服务时使用服务的ip地址
    instance-id: eureka客户端8001
  client:
    service-url:
      defaultZone: http://localhost:9001/eureka/
spring:
  application:
    name: eureka-client-8001

info:
  app.name: eureka-client-8001
  app.author: echo
  company.name: ycl
  build.version: ${project.version}
  build.artifactId: ${project.artifactId}
```
![](https://s1.ax1x.com/2020/05/29/tmmUTP.png)

8002配置同上

![](https://s1.ax1x.com/2020/05/28/tmPkxx.png)
### Eureka自我保护机制
![](https://s1.ax1x.com/2020/05/30/tKBtC6.png)
Eureka在运行期间会统计心跳失败的比例，在15分钟内是否低于85%,如果出现了低于的情况，Eureka Server会将当前的实例注册信息保护起来，同时提示一个警告，一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据。也就是不会注销任何微服务。

默认情况下不建议关闭自我保护机制，如果要关闭
`eureka.server.enable-self-preservation=false`
### eureka 集群搭建
新增子模块eureka-server9002，eureka-server9003
将以上三个服务端互相注册，注⚠️：集群至少是三个起步才叫集群哈
9001 yaml配置
`defaultZone: http://eureka9002.com:9002/eureka,http://eureka9003.com:9003/eureka`
9002 yaml配置 9001 9003,9003 配置9001 9002 这样保证了一个节点在其他节点都有配置

host 配置如下
```
127.0.0.1       eureka9001.com
127.0.0.1       eureka9002.com
127.0.0.1       eureka9003.com
```
我们将8001 注册到9001 注册中心，查看其他两个注册中心是否有同步该注册信息
![](https://s1.ax1x.com/2020/05/30/tK6C0e.png)
可以发现通过集群的配置，单一注册的实例也会注册到其他节点上，这也侧面说明了eureka的ap原则中的高可用性

### eureka和ZK的区别
eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：
- 1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
- 2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
- 3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

Zookeeper 集群中的节点会自动选举一个作为主节点，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这将导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用。

>小结：有了注册中心后，我们的服务间就可以统一注册到一个地方，从而实现服务间的负载均衡调用了；后面将继续总结ribbon


