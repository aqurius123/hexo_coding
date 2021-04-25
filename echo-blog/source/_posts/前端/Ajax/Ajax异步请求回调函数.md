---
title: Ajax
categories:
- 前端
tags: Ajax
date: 2021-04-17
---

> Ajax异步请求时，会涉及到回调函数，原始的Ajax异步请求，异步请求方法和回调函数是分开的两个方法，有的时候为了保证参数在回调函数中保持一致，会将回调函数直接写在Ajax方法内部来直接实现。如：
Ajax.sendAjaxPost(url, params, function callback(){
    
});