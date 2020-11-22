---
title: SpringBoot整合多数据源
categories:
- 后端
tag: 多数据源
date:
---
>不知道你们项目中有没有用到多数据源呢？SpringBoot整合多数据源的方式有哪些呢？我们一起来总结下：

共有三种方式：
- 整合JdbcTemplate
- 整合JPA
- 整合Mybatis

注：三种方式只能选择一种使用，如果你用了mybatis 再定义其他的方式，springboot无法识别该用哪种方式

### JdbcTemplate 多数据源
1. 新建项目，引入依赖

选择开发工具那三个依赖，选择web依赖 选择mysql驱动`MySQL Driver` 选择`JDBC API`

用到的是druid连接池，所以还需要引入对应的依赖

pom依赖如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
    <version>5.1.27</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```
2. 新建数据库连接
```properties
# myStudy 库
spring.datasource.one.url = jdbc:mysql://localhost:3306/myStudy?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
spring.datasource.one.username=root
spring.datasource.one.password=123456
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
# myStudy1 库
spring.datasource.two.url = jdbc:mysql://localhost:3306/myStudy1?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
spring.datasource.two.username=root
spring.datasource.two.password=123456
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
```
3. 新增DataSource数据源配置

因为这里是自定义的数据库连接配置，那么springboot 自动识别就失效了，我们需要手动指定对应的DataSource

新增DataSourceConfig类：
```java
package more.dbs.config;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties(prefix="spring.datasource.one")
    DataSource dbOne(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix="spring.datasource.two")
    DataSource dbTwo(){
        return DruidDataSourceBuilder.create().build();
    }
}
```
新增JDBCTemplateConfig类：
```java
package more.dbs.config;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import javax.sql.DataSource;

@Configuration
public class JDBCTemplateConfig {
    @Bean
    JdbcTemplate jdbcTemplateOne(@Qualifier("dbOne") DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    @Bean
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dbTwo") DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
}
```
4. 添加UserInfo实体，代码略

5. 测试验证
```java
package more.dbs;

import more.dbs.entity.UserInfo;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import java.util.List;

@SpringBootTest
class TestApplicationTests {

    @Test
    void contextLoads() {
    }

    @Autowired
    @Qualifier("jdbcTemplateOne")
    JdbcTemplate jdbcTemplateOne;

    @Autowired
    @Qualifier("jdbcTemplateTwo")
    JdbcTemplate jdbcTemplateTwo;

    @Test
    void testJDBCTemplate(){
        List<UserInfo> queryOne = jdbcTemplateOne.query("select * from user_info", new BeanPropertyRowMapper<>(UserInfo.class));

        List<UserInfo> queryTwo = jdbcTemplateTwo.query("select * from user_info", new BeanPropertyRowMapper<>(UserInfo.class));

        System.out.println(queryOne);
        System.out.println(queryTwo);
    }

}

```
验证结果如下：
![jdbcTemplate_test](https://s1.ax1x.com/2020/04/06/Gs4dqe.png)

### JPA 多数据源
1. 在以上pom 依赖中添加
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
application.propertise中需要加上jpa的配置
```properties
spring.jpa.properties.hibernate.ddl-auto=update
spring.jpa.properties.database-platform=mysql
spring.jpa.properties.database=mysql
spring.jpa.properties.show-sql=true
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL57Dialect
```
2. 因为用了JPA 所以我们的实体要指定表，这里我们新建一个实体`UserInfoJpa`
```java
package more.dbs.entity;

import lombok.Data;
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity(name="user_info")
@Data
public class UserInfoJpa {
    @Id
    private Integer id;

    private String name;

    private String password;

    private Integer deleted;

    @Override
    public String toString() {
        return "UserInfoJpa{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                ", deleted=" + deleted +
                '}';
    }
}
```
3. 新增JPA 数据源bean
新增jpa配置,JpaConfig1代码：
```java
package more.dbs.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import javax.sql.DataSource;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "more.dbs.jpaDao",entityManagerFactoryRef = "factoryBean1",transactionManagerRef = "transactionManager1")
public class JpaConfig1 {
    @Autowired
    @Qualifier("dbOne")
    DataSource dbOne;

    @Autowired
    JpaProperties jpaProperties;

    @Bean
    @Primary
    LocalContainerEntityManagerFactoryBean factoryBean1(EntityManagerFactoryBuilder builder){
        return builder.dataSource(dbOne)
                .properties(jpaProperties.getProperties())
                .persistenceUnit("jpa_db1")
                .packages("more.dbs.entity")
                .build();
    }
    // 配置事务
    @Bean
    PlatformTransactionManager transactionManager1(EntityManagerFactoryBuilder builder){
        return new JpaTransactionManager(factoryBean1(builder).getObject());
    }
}
```
同理，新增JpaConfig2，将1改成2，dbOne 改成 dbTwo即可,代码略

注⚠️：这个地方必须加上@Primary注解，表示当有多个LocalContainerEntityManagerFactoryBean 优先使用加了此注解的bean,同样需要在DataSourceConfig类中的dbOne()上加上此注解；

新建jpaDao 中创建两个dao接口 继承自JpaRepository<UserInfoJpa,Integer>

UserInfoDao1代码如下,UserInfoDao2同UserInfoDao1：
```java
package more.dbs.jpaDao;

import more.dbs.entity.UserInfoJpa;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserInfoDao1 extends JpaRepository<UserInfoJpa,Integer> {

}
```
4. 测试
新增测试代码如下：
```java
@Autowired
UserInfoDao1 userInfoDao1;

@Autowired
UserInfoDao2 userInfoDao2;

@Test
void testJpa(){
    List<UserInfoJpa> userInfoList1 = userInfoDao1.findAll();
    List<UserInfoJpa> userInfoList2 = userInfoDao2.findAll();

    System.out.println(userInfoList1);
    System.out.println(userInfoList2);
}
```
![jpa_test](https://s1.ax1x.com/2020/04/06/Gy9YWV.png)

### MyBatis 多数据源
1. 新增pom依赖：
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```
2. 新增mybatis 数据源配置

MybatisConfig1代码如下：
```java
package more.dbs.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.Resource;
import javax.sql.DataSource;

@Configuration
@MapperScan(basePackages = "more.dbs.mybatisMapper",sqlSessionFactoryRef = "sqlSessionFactory1",sqlSessionTemplateRef = "sqlSessionTemplate1")
public class MybatisConfig1 {
    @Resource(name = "dbOne")
    DataSource dbOne;

    @Bean
    SqlSessionFactory sqlSessionFactory1() {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        try {
            bean.setDataSource(dbOne);
            return bean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    @Bean
    SqlSessionTemplate sqlSessionTemplate1() {
        return new SqlSessionTemplate(sqlSessionFactory1());
    }
}
```
同理，增加MybatisConfig2 配置，将1改成2，dbOne改成dbTwo即可

3. 新增dao 和 mapper 层

mybatisMapper 下新增UserInfoMapper1 和UserInfoMapper1.xml

UserInfoMapper1 代码：
```java
package more.dbs.mybatisMapper;

import more.dbs.entity.UserInfo;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface UserInfoMapper1 {
    List<UserInfo> getAllUserInfo1();
}
```
UserInfoMapper1.xml 代码：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="more.dbs.mybatisMapper.UserInfoMapper1">

    <select id="getAllUserInfo1" resultType="more.dbs.entity.UserInfo">
        select * from user_info
    </select>
</mapper>
```
同理在mybatisMapper 下新增UserInfoMapper2 和UserInfoMapper2.xml，代码略

4. 测试
```java
@Autowired
UserInfoMapper1 userInfoMapper1;
@Autowired
UserInfoMapper2 userInfoMapper2;

@Test
void testMybatis() {
    List<UserInfo> userInfoList1 = userInfoMapper1.getAllUserInfo1();
    List<UserInfo> userInfoList2 = userInfoMapper2.getAllUserInfo2();

    System.out.println(userInfoList1);
    System.out.println(userInfoList2);
}
```
![mybatis_test](https://s1.ax1x.com/2020/04/06/GyYbOU.png)

> 小结： 至此springBoot整合多数据源的三种配置方式总结完了，其实工作中很少这么玩儿，基本都是分库分表拆分业务了。
但是如果有些需求真的要是如此的话，希望我们也能找到对应的办法！