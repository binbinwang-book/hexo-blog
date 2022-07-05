---
title: Jsp和Servlet有什么区别？
date: 2022-03-12 17:47
tags: [Jsp,Servlet]
categories: [后台]
---

# Jsp和Servlet有什么区别？

## 1、不同之处在哪？

- Servlet在Java代码中通过HttpServletResponse对象动态输出HTML内容
- JSP在静态HTML内容中嵌入Java代码，Java代码被动态执行后生成HTML内容

## 2、各自的特点

- Servlet能够很好地组织业务逻辑代码，但是在Java源文件中通过字符串拼接的方式生成动态HTML内容会导致代码维护困难、可读性差
- JSP虽然规避了Servlet在生成HTML内容方面的劣势，但是在HTML中混入大量、复杂的业务逻辑同样也是不可取的

## 3、通过MVC双剑合璧

既然JSP和Servlet都有自身的适用环境，那么能否扬长避短，让它们发挥各自的优势呢？答案是肯定的——MVC(Model-View-Controller)模式非常适合解决这一问题。

MVC模式（Model-View-Controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）：

- Controller——负责转发请求，对请求进行处理
- View——负责界面显示
- Model——业务功能编写（例如算法实现）、数据库设计以及数据存取操作实现

在JSP/Servlet开发的软件系统中，这三个部分的描述如下所示：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07m68pbcxj21400figmq.jpg)

1. Web浏览器发送HTTP请求到服务端，被Controller(Servlet)获取并进行处理（例如参数解析、请求转发）
2. Controller(Servlet)调用核心业务逻辑——Model部分，获得结果
3. Controller(Servlet)将逻辑处理结果交给View（JSP），动态输出HTML内容
4. 动态生成的HTML内容返回到浏览器显示

MVC模式在Web开发中的好处是非常明显，它规避了JSP与Servlet各自的短板，Servlet只负责业务逻辑而不会通过out.append()动态生成HTML代码；JSP中也不会充斥着大量的业务代码。这大大提高了代码的可读性和可维护性。



