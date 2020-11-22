---
title: springboot集成mybatis和druid监控
categories:
- 后端
tags: springboot,mybatis,druid
date:
---
>springboot操作数据的库的方式有很多，jdbcTemplate,jpa,集成mybatis...现在的日常开发，几乎都是采用mybatis框架;它灵活而又层次分明的设计极大的简化了我们对数据库的操作！

概念：
- MyBatis是一流的持久性框架,mybatis支持自定义SQL，存储过程和高级映射。MyBatis消除了几乎所有的JDBC代码以及参数的手动设置和结果检索。MyBatis可以使用简单的XML或注释进行配置，并将图元，映射接口和Java POJO（普通的旧Java对象）映射到数据库记录.官网：<a href="https://mybatis.org/mybatis-3/" target="_blank">mybatis</a>

- druid 德鲁伊是阿里巴巴的开源组件之一，结合了C3P0，DBCP的优点，并且自带日志监控！Druid 可以天然的监控 SQL 和 数据库连接池的状况！wiki: <a href="https://github.com/alibaba/druid/wiki" target="_blank">druid wiki</a>
看下它的配置属性列表：

| 配置     | 缺省值   | 说明    |
| ----------------------------------------- | ------------------ | ------------------------------------------------------------ |
| name                                      |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this). 另外配置此属性至少在1.0.5版本中是不起作用的，强行设置name会出错。[详情-点此处](http://blog.csdn.net/lanmo555/article/details/41248763)。 |
| url                                       |                    | 连接数据库的url，不同数据库不一样。例如： mysql : jdbc:mysql://10.20.153.104:3306/druid2 oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                                  |                    | 连接数据库的用户名                                           |
| password                                  |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。[详细看这里](https://github.com/alibaba/druid/wiki/使用ConfigFilter) |
| driverClassName                           | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName |
| initialSize                               | 0                  | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                                 | 8                  | 最大连接池数量                                               |
| maxIdle                                   | 8                  | 已经不再使用，配置了也没效果                                 |
| minIdle                                   |                    | 最小连接池数量                                               |
| maxWait                                   |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements                    | false              | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxPoolPreparedStatementPerConnectionSize | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery                           |                    | 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。 |
| validationQueryTimeout                    |                    | 单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法 |
| testOnBorrow                              | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                              | false              | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testWhileIdle                             | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| keepAlive                                 | false （1.0.28）   | 连接池中的minIdle数量以内的连接，空闲时间超过minEvictableIdleTimeMillis，则会执行keepAlive操作。 |
| timeBetweenEvictionRunsMillis             | 1分钟（1.0.14）    | 有两个含义： 1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun                    | 30分钟（1.0.14）   | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis                |                    | 连接保持空闲而不被驱逐的最小时间                             |
| connectionInitSqls                        |                    | 物理连接初始化的时候执行的sql                                |
| exceptionSorter                           | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                                   |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat 日志用的filter:log4j 防御sql注入的filter:wall |
| proxyFilters                              |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |


### 创建project
选择Spring Initializr初始化项目，说一下选择依赖这里：Developer Tools-> 我一般三个都勾选(免得后面自己手动添加麻烦)，Web 勾选上Spring Web就可以了。因为我们这里选择用mysql，所以SQL 这里我们选择MySQL Driver;最后项目生成后别忘了选择Enable Atuo Import，选择自动导入依赖！

选择Idea开发我一般都会连上数据库，选择右侧Database链接好数据源，这样就不用每次都打开数据库看表了！第一次设置要下载链接驱动包，这里按常识来就好不再赘述！下载好测试链接：
![](https://s1.ax1x.com/2020/03/18/8wYOht.png)
![](https://s1.ax1x.com/2020/03/18/8wYxc8.png)

### druid配置
添加pom依赖：
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```
新建application.yaml 增加如下数据源和mybatis配置：
```yaml
spring:
  datasource:
    username: root
    password: 123456
    # 注意8.0以上需要时区的配置
    url: jdbc:mysql://xxx.xxx.xxx.xxx:3306/usertest?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver

    type: com.alibaba.druid.pool.DruidDataSource
    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错 java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址： https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

# 实体类和mapper配置
mybatis:
  type-aliases-package: com.springstudy.entity
  mapper-locations: classpath:mapper/*/*.xml
  configuration:
    #当查询数据为空时字段返回为null，不加这个查询数据为空时，字段将被隐藏
    call-setters-on-nulls: true
```
新增配置类DruidConfig:
```java
package com.springstudy.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;

/**
 * druid配置
 */
@Configuration
public class DruidConfig {

    // 绑定配置的bean
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDateSource() {
        return new DruidDataSource();
    }

    // 注册后台监控页面。SpringBoot 如何注册Servlet
    // 没有web.xml 的情况配置 Servlet 的方法:ServletRegistrationBean
    // 测试访问 /druid
    @Bean
    public ServletRegistrationBean statViewServlet() {
        // StatViewServlet 配置后台监控
        ServletRegistrationBean bean = new ServletRegistrationBean(new
                StatViewServlet(), "/druid/*");
        HashMap<String, String> map = new HashMap<>();
        //后台的登录用户名和密码
        map.put("loginUsername", "admin");
        map.put("loginPassword", "123456");
        // 访问权限
        // map.put("allow","localhost"); //只允许本机访问
        map.put("allow", ""); // 所有人都可以访问
        // deny拒绝访问
        // map.put("deny","192.168.1.1"); // ip会被拒绝访问
        bean.setInitParameters(map); //设置servlet的初始化参数
        return bean;
    }

    // 过滤器的配置，看看哪些请求需要被过滤
    // 没有web.xml 的情况配置 Filter 的方法！ FilterRegistrationBean
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        // 配置内容
        // 配置哪些请求可以被过滤！
        HashMap<String, String> map = new HashMap<>();
        map.put("exclusions", "*.js,*.css,/druid/*");
        bean.setInitParameters(map);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```
启动项目访问：localhost:8080/druid 输入用户名密码
![](https://s1.ax1x.com/2020/03/18/8wYv1f.png)
### 新增接口测试
代码省略。。。
访问http://localhost:8080/test/queryAllUser
查看druid 监控：
![](https://s1.ax1x.com/2020/03/18/8wYLtI.png)

>至此，spingboot集成mybatis和druid sql监控界面完毕，你就可以放开手脚的实现你的业务代码了！Just do it!
