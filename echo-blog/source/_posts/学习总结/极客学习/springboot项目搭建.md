---
title: springboot项目搭建
categories: 极客学习
- 学习总结
tags: springboot
date: 20210516
---

> 说明：在极客学习过程中，搭建微博话题项目时，发现了自己之前没有深入了解以及遗忘了的知识点，在这里做一下总结及回顾。

## springmvc原理
1. 在springmvc框架中，当接口返回为字符串时，mvc默认认为最终返回的是约定好的页面文件。
- 总结：所以如果你是想简单的返回一个字符串的话，必须在接口上添加@ResponseBody注解进行封装处理。

## springboot处理静态文件
> 首先，这里必须纠正一下自己之前的认知，静态文件的访问并不是以接口的方式来进行访问的。而是直接通过配置文件中对静态文件的路径配置以及方法中返回的。如下：

~~~
public ModelAndView getStatic() {
//        ModelAndView modelAndView = new ModelAndView();
//        modelAndView.addObject("test");
        ModelAndView modelAndView = new ModelAndView("test.html");
        return modelAndView;
    }
注意：在浏览器直接通过http://localhost:端口/static/test.html即可访问指定的静态文件了
~~~



