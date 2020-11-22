---
title: 初窥SpringSecurity安全框架
categories:
- 后端
tags: Security
date: 
---
>作为一名开发怎能不知道大名顶顶的安全框架呢？市面上流行的安全框架有：shiro和springSecurity。那么你经常用哪个框架做安全访问控制呢？因为SpringBoot集成了SpringSecurity,所以我们这次来聊聊它
### 概念：
Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI（控制反转Inversion of Control ,DI:Dependency Injection 依赖注入）和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。

SpringSecurity官网：<a href="https://spring.io/projects/spring-security" target="_blank">官网</a>
![](https://s1.ax1x.com/2020/03/28/GkT0MT.png)

### 对应依赖
pom依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
SpringSecurity包含的模块:
![](https://s1.ax1x.com/2020/03/28/GkHLKf.png)
继续我们可以看到这么一幅图，可以知道它是基于servlet过滤器实现的：
![](https://s1.ax1x.com/2020/03/28/Gkbgij.png)
http配置相关：
![](https://s1.ax1x.com/2020/03/28/GkOYoF.png)
用户密码验证：
![](https://s1.ax1x.com/2020/03/28/GAOvK1.png)
### 创建项目
1.创建新的项目，选择thymelaef和SpringWeb依赖
2.新增静态资源(代码省略，主要是各角色页面，需要此部分代码可私我)
3.添加路由控制页面跳转
```java
package com.springstudy.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class RouterController {
    @RequestMapping({"/","/index"})
    public String index(){
        return "index";
    }

    @RequestMapping("/toLogin")
    public String toLogin(){
        return "views/login";
    }

    // 通过restful 实现复用
    @RequestMapping("/level1/{id}")
    public String level1(@PathVariable("id") int id){
        return "views/level1/"+id;
    }

    @RequestMapping("/level2/{id}")
    public String level12(@PathVariable("id") int id){
        return "views/level2/"+id;
    }

    @RequestMapping("/level3/{id}")
    public String level3(@PathVariable("id") int id){
        return "views/level3/"+id;
    }
}
```
4.启动项目查看页面效果（默认登陆用户名是user,登陆密码在控制台）
![](https://s1.ax1x.com/2020/03/29/GEMfD1.png)
后面我们要实现对应的权限控制效果！

### 自定义登陆用户和密码
老是这样登不行啊，我么来自定义配置一下登陆用户名和密码
```properties
# 清除thymeleaf缓存 这使得我们改动html代码不用重启项目 build一下即可生效
spring.thymeleaf.cache=false
# 用户名
spring.security.user.name=admin
# 密码
spring.security.user.password=123456
```
### 新增SecurityConfig配置类
```java
package com.springstudy.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // 可以再这里导入userService相关配置
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 不同用户访问内容不同！ 这里，过滤器，登录注销规则，安全配置，OAuth2配置
        // 我们平时只需要配置一些基本的规则即可！

        // 首页是允许所有人访问的！
        // 定制授权规则： 那些请求，哪些人可以访问！
        http.authorizeRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasRole("guest")
                .antMatchers("/level2/**").hasRole("vip")
                .antMatchers("/level3/**").hasRole("svip");

        // 自定义登录页 配置后默认登陆页将失效
        // login跳转到登录页  /login?error 登录失败
        http.formLogin()
                .usernameParameter("username")
                .passwordParameter("password")
                .loginPage("/toLogin")
                .loginProcessingUrl("/login"); // 登陆表单提交请求！

        // 如果注销404，因为 Security 默认是防止 csrf 跨站伪请求！
        // http.csrf().disable(); // 可能会让我们系统不安全

        // 注销 开启默认的注销功能！
        http.logout().logoutSuccessUrl("/"); // 注销成功后跳转至首页！

        // 自定义的登录页需要配置 rememberMe 的参数名,就可以绑定到我们前端的！
        // 记住我功能
        http.rememberMe().rememberMeParameter("remember");
    }

    // 定义用户的认证规则 （角色，密码....）
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 定义user1 uer2 user3 三个用户的角色权限
        // 这里我们一般在用户角色表里存储用户的角色信息
        // 密码没有加密会报500错误，这里我们只定义了用户的密码加密，但是没有定义用户的认证规则的加密方式
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                // 一个人可以拥有多个角色！
                .withUser("user1")
                .password(new BCryptPasswordEncoder().encode("123456"))
                .roles("guest")
                .and()
                .withUser("user2")
                .password(new BCryptPasswordEncoder().encode("123456"))
                .roles("guest","vip")
                .and()
                .withUser("user3")
                .password(new BCryptPasswordEncoder().encode("123456"))
                .roles("guest","vip","svip");
    }
}
```
### 修改前台配置
```html
<!DOCTYPE html>
<html lang="en"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>首页</title>
    <!--semantic-ui-->
    <link href="https://cdn.bootcss.com/semantic-ui/2.4.1/semantic.min.css" rel="stylesheet">
    <link th:href="@{/test/css/qinstyle.css}" rel="stylesheet">
</head>
<body>

<!--主容器-->
<div class="ui container">

    <div class="ui segment" id="index-header-nav" th:fragment="nav-menu">
        <div class="ui secondary menu">
            <a class="item"  th:href="@{/index}">首页</a>
            <!--登录注销-->
            <div class="right menu">
                 <!--核心类：Authentication-->
                <!-- 如果未登录就显示登陆按钮 -->
                <div sec:authorize="!isAuthenticated()">
                    <a class="item" th:href="@{/toLogin}">
                        <i class="address card icon"></i> 登录
                    </a>
                </div>

                <!-- 如果已登录，显示用户的信息 -->
                <div sec:authorize="isAuthenticated()">
                    <a class="item">
                        <!--<i class="address card icon"></i>-->
                        用户名：<span sec:authentication="principal.username"></span> &nbsp;
                        角色：<span sec:authentication="principal.authorities"></span>
                    </a>
                </div>

                <div sec:authorize="isAuthenticated()">
                    <!-- 将注销请求也改成post提交即可！ -->
                    <form th:action="@{/logout}" method="post">
<!--                        <button type="submit" class="def-log-out">注销</button>-->
                        <a class="item" th:href="@{/logout}" style="text-decoration:underline;">
                            注销
                        </a>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <div class="ui segment" style="text-align: center">
        <h3>Spring Security Study</h3>
    </div>

    <div>
        <br>
        <div class="ui three column stackable grid">
<!--            <div sec:authorize="hasRole('vip1')">-->
                <div class="column" sec:authorize="hasRole('guest')">
                    <div class="ui raised segment">
                        <div class="ui">
                            <div class="content">
                                <h5 class="content">Level 1</h5>
                                <hr>
                                <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                                <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                                <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
                            </div>
                        </div>
                    </div>
                </div>
<!--            <div sec:authorize="hasRole('vip2')">-->
                <div class="column" sec:authorize="hasRole('vip')">
                    <div class="ui raised segment">
                        <div class="ui">
                            <div class="content">
                                <h5 class="content">Level 2</h5>
                                <hr>
                                <div><a th:href="@{/level2/1}"><i class="bullhorn icon"></i> Level-2-1</a></div>
                                <div><a th:href="@{/level2/2}"><i class="bullhorn icon"></i> Level-2-2</a></div>
                                <div><a th:href="@{/level2/3}"><i class="bullhorn icon"></i> Level-2-3</a></div>
                            </div>
                        </div>
                    </div>
                </div>
<!--            <div sec:authorize="hasRole('vip3')">-->
                <div class="column" sec:authorize="hasRole('svip')">
                    <div class="ui raised segment">
                        <div class="ui">
                            <div class="content">
                                <h5 class="content">Level 3</h5>
                                <hr>
                                <div><a th:href="@{/level3/1}"><i class="bullhorn icon"></i> Level-3-1</a></div>
                                <div><a th:href="@{/level3/2}"><i class="bullhorn icon"></i> Level-3-2</a></div>
                                <div><a th:href="@{/level3/3}"><i class="bullhorn icon"></i> Level-3-3</a></div>
                            </div>
                        </div>
                    </div>
                </div>
        </div>
    </div>
</div>
<script th:src="@{/test/js/jquery-3.1.1.min.js}"></script>
<script th:src="@{/test/js/semantic.min.js}"></script>
</body>
</html>
```
### 重启项目验证
![](https://s1.ax1x.com/2020/03/29/GV10wn.png)
可以看到3个用户，可以看到不同的模块，实现了权限的控制。
注：在我们配置注销时，可以看到springSecurity已经帮我们配置好了
![](https://s1.ax1x.com/2020/03/29/GV3g4P.png)
当然我们实际的处理方式是不同的角色配置不同的菜单，不会这么控制！这里只是举例说明Security权限控制以及thymeleaf权限控制语法
### 登陆页配置记住我
>功能:用户没有登录的时候，就只显示导航栏!如果登录了，只显示自己权限能够看到的东西 根据不同的权限，前端展示不同的功能!

springsecrity 和 thymealef 结合:

添加依赖：
```xml
<!-- https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras- springsecurity5 -->
<dependency>
    <groupId>org.thymeleaf.extras</groupId> 
    <artifactId>thymeleaf-extras-springsecurity5</artifactId> 
    <version>3.0.4.RELEASE</version>
</dependency>
```
前端代码：
```html
<input type="checkbox" name="remember"> 记住我
```
后端代码：
```java
 // 自定义的登录页需要配置 rememberMe 的参数名,就可以绑定到我们前端的! 
 // 记住我功能 
 http.rememberMe().rememberMeParameter("remember");

 // 定制登陆页
 http.formLogin()
    .usernameParameter("username") // 前端用户名提交参数 
    .passwordParameter("password") // 前端密码提交参数 
    .loginPage("/toLogin") // 登录页面 
    .loginProcessingUrl("/login"); // 登陆表单提交请求!
```
启动测试看用户信息是否存如cookie:
![](https://s1.ax1x.com/2020/03/29/GVY7sU.png)
可以删除cookie后刷新页面看看是否自动退出了！

### 退出的问题
点击退出发现报错了
![](https://s1.ax1x.com/2020/03/29/GVY7sU.png)

这是为什么呢？
原来SpringSecurity禁止了get方式的退出,以防止 csrf 跨站伪请求！
```java
// 如果注销404，因为 Security 默认是防止 csrf 跨站伪请求！
// http.csrf().disable(); // 可能会让我们系统不安全
```
那我们把退出操作改成表单提交的post方式请求即可；

修改index.html注销代码：
```html
<form th:action="@{/logout}" method="post">
    <button type="submit" class="def-log-out">注销</button>
    <!-- <a class="item" th:href="@{/logout}" style="text-decoration:underline;">
        注销
    </a> -->
</form>
```
build 后再次点击注销可以验证下！

>小结：Spring集成Security后，sercurity就变得轻巧了很多，且因为功能强大所以在SpringBoot中面对shrio优势明显，
像我们常见的功能：单点登陆，微信qq登陆认证等它都作了对应的支持！本次只是分享Security的初步使用!
