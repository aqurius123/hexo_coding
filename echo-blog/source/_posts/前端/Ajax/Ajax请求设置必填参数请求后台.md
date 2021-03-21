---
title: ajax
categories:
- 前端
tags: ajax
date:
---

> 使用场景：前后端联调时，前端必须带有token才能访问后台(后台要对登录用户进行校验，是否已登录),后台是自己在项目中写的后台，对前端不熟，所以自己只能单独写了一个html文件，使用ajax测试联调后台；
下面是解决方式，有两种：

- 方法一：beforeSend方法：
~~~
$("#test").click(function() {
     $.ajax({
         type: "GET",
         url: "default.aspx",
         beforeSend: function(request) {
            request.setRequestHeader("Sin-Token","zzz");
         },
         success: function(result) {
             alert(result);
         }
     });
 });
~~~
- 方法二：setting参数 headers
~~~
$.ajax({
    headers: {
        "Sin-Token": "zzz"
    },
    type: "get",
    success: function (data) {
    }
});
~~~

经过测试，这两种方法都可以解决自己的问题。
原文链接：https://blog.csdn.net/zzk220106/article/details/81316092

