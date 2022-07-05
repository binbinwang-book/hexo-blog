---
title: Spring MVC架构破壁
date: 2022-03-12 17:47
tags: [Srping,MVC]
categories: [后台]
---

# Spring MVC架构破壁

## 一、MVC 简介

### （一）MVC：开发Web应用的通用架构方式

Front Controller（MVC）：前端控制器

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077dichg0j21es0suq7u.jpg)

```
>> 首先用户端的请求通过HTTP协议到达 Front Controller ，Front Controller 了解这个请求应当被谁处理
>> Front Controller 将请求转给相关的 Controller 处理业务逻辑，Controller处理业务逻辑后将业务数据返回给  Front Controller
>> 此时前端控制器将 Model数据 分发给业务视图 View template，由业务生成业务界面，并将业务界面数据返回给 Front Controller
>> 最后 Front Controller 返回给请求方
```

### （二）MVC 类似分诊台

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077d3xproj219d0i1761.jpg)

### （三）View 视图层

为用户提供UI，重点关注数据的呈现

### （四）Model 模型层

业务数据的信息表示，关注支撑业务的信息构成，通常是多个业务实体的组合。

### （五）Controller 控制器

控制层调用业务逻辑产生合适的model，同时传递数据给视图层用于呈现。

### （六）小结

MVC 是一种架构模式：程序分层、分工合作、既相互独立，又协同工作

MVC 是一种思考方式，需要将什么信息展示给用户？如何布局？调用哪些业务逻辑？

## 二、Spring MVC基本概念

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077vz0ptfj20kr0jdq43.jpg)

### （一）DispatcherServlet 

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077iwt8v9j20nr0l9gmk.jpg)

### （二）Controller 

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077jyxmhwj212206p74o.jpg)

### （三）HandlerAdapter 适配器模式

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077ldoahcj21ar0l8ta9.jpg)

### （四）HandlerInterceptor 拦截器

支持配置三个方法：
- afterCompletion
- postHandle
- preHandle

可以在调用 Controller 之前、之后都进行一些行为。

### （五）HandlerMapping 前端控制器和Controller映射关系

1. Help DispatcherServlet to get the right controller

2. Wrap controller with HandlerInterceptor

### （六）HandlerExecutionChain

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077t24uv4j21ho0i90ul.jpg)

### （七）ViewResolver 视图解析器

Help DispatcherServlet to Resolve the right View to render page.

### （八）小结

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h077yelahij21hw0t1q74.jpg)

主要要写的是：Controller

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07840mpn5j20zg0tztbi.jpg)

## 三、Maven

### （一）POM：Project Object Model

xml文件，完成依赖管理。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0788u0nfrj20wq0obn01.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h078fdznr3j21ge0r8aez.jpg)

### （二）Coordinates

Maven 通过坐标实现依赖的功能：
- groupID
- artifactId
- version
- packaging

通过坐标实现唯一的定位。
