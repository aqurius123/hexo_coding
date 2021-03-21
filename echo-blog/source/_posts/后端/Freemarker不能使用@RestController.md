---
title: Freemarker
categories:
- 后端
tags: Freemarker
date:
---

1、为何在controller控制层使用@RestController注解竟然无效呢？但是 
   @Controller却可以。。。。
2、在定义的接口的方法中的参数中一定要有参数类型-----一般都是Map,否则会 
   报错：
   如：--------这会报错
   ~~~
    @RequestMapping("/name")
    public String getName(Map map) {//并没有使用传递的map，会报错
        Map map2 = new HashMap();
        map2.put("name", "柯帅");
        //model.addAttribute("name", "柯帅");
        return "demo";
    }
~~~

正确如下：
~~~
  @RequestMapping("/name")
    public String getName(Map map) {
        map.put("name", "柯帅");
        //model.addAttribute("name", "柯帅");
        return "demo";
    }