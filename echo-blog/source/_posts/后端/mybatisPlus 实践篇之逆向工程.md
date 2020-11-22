---
title: mybatisPlus 实践篇之逆向工程
categories:
- 后端
tags: mybatis-plus
date:
---
> 初次听说mybatis-plus感觉这东西取名字都很有意思，像极了现在的iPhone 到iPhone xplus;不得不说水果公司真的引领了很多的“潮流”啊；
最近公司的一个新的项目，用的也是mybatis-plus,但是用的感觉不是那么好，所有就有了这篇实践！其实了解了也就和JPA差不多，废话不多说我们开始吧！

### 概念
Mybatis-Plus（简称MP）是一个 Mybatis 的增强工具，在 Mybatis 的基础上只做增强不做改变，为简化开发、提高效率而生.
其实要我说，最大的亮点就是它的逆向工程和去mapper.xml了,但是也仅是针对单表，多表还是需要mapper.xml的，后面我们都会给出示例。

官网：<a href="https://mp.baomidou.com/" target="_blank">MyBatis-Plus</a>

### 特性：
- **无侵入**: 只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑

- **损耗小**: 启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作

- **强大的 CRUD 操作**: 内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求. 继承一个类就好了!

- **支持 Lambda 形式调用**: 通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错

- **支持主键自动生成**: 支持多达 4 种主键策略(内含分布式唯一 ID 生成器 - Sequence)，可自由配 置，完美解决主键问题

- **支持 ActiveRecord 模式**: 支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大 的 CRUD 操作

- **支持自定义全局通用操作**: 支持全局通用方法注入( Write once, use anywhere )

- **内置代码生成器** (pojo、dao、xml、service、.....):采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来 使用

- **内置分页插件**: 基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同 于普通 List 查询
分页插件

- **支持多种数据库**: 支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、 Postgre、SQLServer 等多种数据库

- **内置性能分析插件**: 可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢 查询

- **内置全局拦截插件**: 提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误 操作

### 新建Spring项目，引入依赖
```xml
 <!--导入 对应的依赖 这里用3.0.5 官网上最新的是一个临时版temp的-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.0.5</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- 模板引擎 记得加上不然后面自动生成代码会报错-->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.0</version>
</dependency>
<!--swagger 依赖 记得要加上，不然后面自动生成实体代码注解会爆红-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.9.6</version>
</dependency>
```
提一下：spring 2.0x都是配置的mysql8的驱动，8.0以上的驱动需要配置数据库时区！

另外，注意官网上的这句话：
![](https://s1.ax1x.com/2020/05/09/Y1V5Ox.jpg)

### yaml配置
```yaml
spring:
  application:
    name: test-mbp
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mp_study?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
    username: root
    password: 123456
```
配完这些记得启动一下项目，看看是否有问题，及时处理掉报错！

### 配置代码生成器
```java
package com.demo;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.po.TableFill;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

import java.util.ArrayList;

public class TestCodeGenerator {
    public static void main(String[] args) {
        // 模块名
        String moduleName = "study";

        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        // 获取当前项目根路径
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath+"/src/main/java");
        gc.setAuthor("Echo");
        gc.setOpen(false); //不打开生产的文件
        gc.setFileOverride(false); //不覆盖之前生成的文件
        gc.setServiceName("%Service");
        gc.setIdType(IdType.AUTO);// 主键策略 自增  注意要和数据库中表实际情况对应
        gc.setDateType(DateType.ONLY_DATE);
        gc.setSwagger2(true);//自动开启swagger2的支持
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUrl("jdbc:mysql://localhost:3306/mp_study?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(moduleName);
        pc.setParent("com.demo");
        pc.setController("controller");
        pc.setService("service");
        pc.setEntity("entity");
        pc.setMapper("mapper");
        mpg.setPackageInfo(pc);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        // strategy.setInclude("t_user");
        //可以用同配符号:表示生成t_开头的对应库下所有表
        strategy.setInclude("t"+"_\\w*");
        strategy.setNaming(NamingStrategy.underline_to_camel);// 下划线转驼峰
        strategy.setTablePrefix("t_");//去掉t_这个前缀后生成类名
        strategy.setEntityLombokModel(true);// 自动生成lombok注解  记住要有lombok依赖和对应的插件哈
        strategy.setLogicDeleteFieldName("is_deleted");//设置逻辑删除字段 要和数据库中表对应哈

        // 设置创建时间和更新时间自动填充策略
        TableFill created_date = new TableFill("created_date", FieldFill.INSERT);
        TableFill updated_date = new TableFill("updated_date", FieldFill.INSERT_UPDATE);
        ArrayList<TableFill> tableFills = new ArrayList<>();
        tableFills.add(created_date);
        tableFills.add(updated_date);
        strategy.setTableFillList(tableFills);

        // 乐观锁策略
        strategy.setVersionFieldName("version");
        strategy.setRestControllerStyle(true);//采用restful 风格的api
        strategy.setControllerMappingHyphenStyle(true); // controller 请求地址采用下划线代替驼峰
        mpg.setStrategy(strategy);

        // 执行
        mpg.execute();
    }
}
```
当然代码生成器中写死的东西我们可以封装下，便于后面项目多的时候调用！或者项目中可以集成这些配置项到ui中，所有的东西都有实施人员来配置，调用接口直接生成到对应的基础框架代码中就好了！
题外话：bootdo 之前有用过，基础框架搭好后就是前端选择表和创建菜单后，可以基于表生成对应的基础的前后端的代码！
### 运行验证
![](https://s1.ax1x.com/2020/05/10/Y1GcKU.jpg)
可以看到在我们的包下生成来对应的各种包，这里我只建来`t_user`这一张表来演示；注意代码生成器里的逻辑都是基于表的配置来写的，要保持一致不然会报错！

### 验证swagger
启动验证一下
![](https://s1.ax1x.com/2020/05/10/Y1NeJA.jpg)
呃呃呃，没扫描到mapper包,那我们去加一个包扫描即可！
`@MapperScan("com.demo.study.mapper")`

我们去看对应的mapper 发现没有@Mapper注解，说明这东西还有点没完善哈，按理说其他的层的注解你都生成了，mapper注解没生成有点那啥了，哈哈！
注意加上`@EnableSwagger` 它开启了Swagger的注解是针对生成的代码的，启动类上还是得自己加！
另外记得加上咱们之前****Swagger食用详解**** 中对应swagger的配置哈。

![](https://s1.ax1x.com/2020/05/10/Y1UDjP.jpg)

> 小结：好了，mp的逆向工程使用说明就先说到这儿，接下来mp的crud操作，统一接口返回和异常封装准备再写一篇示例，加油！


