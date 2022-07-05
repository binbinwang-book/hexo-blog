---
title: 《SpringBoot + Vue3教程》（二）Spring Boot项目搭建
date: 2021-12-12 16:25
tags: [SpringBoot,Vue3]
categories: [后台]
---

# 《SpringBoot + Vue3教程》（二）Spring Boot项目搭建

## 一、SpringBoot项目目录

做项目就像建房子，要先把地基打好，然后开始开发。

SpringBoot不需要配置容器，是因为使用了嵌入式容器，默认使用tomcat启动，默认端口8080。

SpringBoot项目使用main函数启动，一般放在XXXApplication类里，需要加 @SpringBootApplication 注解

Maven Wrapper可以不需要提前下载好 Maven，由它去下载Maven。

一些自动化打包的场景可以用 Maven Wrapper。
流水线：自动拉代码，自动下载Maven，自动编译打包。

## 二、项目初始配置

### 1. 遇到的问题：新建项目后，无法运行

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb1fd4i6nj30r206m3yp.jpg)

Debug过程：

Google发现和maven有关系，发现自己电脑里不还没装 maven， 于是乎：

>>> brew install maven

安装完成后，项目即可run起来：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb75ktompj30jw05gaa1.jpg)

我们访问 http://localhost:8080/

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb7aroo1jj3112096t9k.jpg)

发现已经连接上。

### 2. 在工程中所有用到的语言都改为 utf-8 ：

把所有看到的 CBK 都改成 utf-8

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb1fvut27j30c205gwej.jpg)

### 3. Maven 具体有什么用？

我先不说maven，也不说java开发，先说做菜，你可能像做个红烧小排(HongshaoxiaopaiApp)，你需要的材料是：

1. 小排(xiaopai.jar)，要小猪的（version=little pig）。
2. 酱油(jiangyou.jar)，要82年的酱油（version=1982）
3. 盐(yan.jar)
4. 糖(tang.jar)，糖要广东产的（version=guangdong）
5. 生姜(shengjiang.jar)
6. 茴香(huixiang.jar)

于是，你要去菜场买小排，去门口杂货店买酱油，买盐……可能你家门口的杂货店还没有1982年的酱油，你要去3公里外的农贸市场买……你买原材料的过程估计会很痛苦，可能买到的材料不是1982年的，会影响口感。

在你正式开始做小排前，你会为食材的事情，忙得半死。

现在有个超市出了个盒装版的半成品红烧小排，把生的小排，1982年的酱油，盐，广东产的糖等材料打包成一个盒子里，你回家只要按照说明，就能把红烧小排做出来，不用考虑材料的来源问题。

Maven就是那个超市，红烧小排就是你要开发的软件，酱油、盐什么的就是你开发软件要用到的jar包——我们知道，开发java系统，下载一堆jar包依赖是很正常的事情。有了maven，你不用去各个网站下载各种版本的jar包，也不用考虑这些jar包的依赖关系。Maven会给你搞定，就是超市的配菜师傅会帮你把红烧小排的配料配齐一样。

现在你应该明白Maven是做什么的了吧。

### 4.文件工程要交给git来管理

### 5.可以通过 .gitignore 来忽略文件，本地工作空间相关的文件不要提交，比如： .idea、target

### 6. 启动日志优化

#### （1）logback日志样式修改

错误的日志会创建到相关的目录下，因为后台很大概率很难复现，所以需要error要保存到 .log 中，方便查询问题。

## 三、创建一个 hello world接口

### 1. 创建接口

接口一般是放在 controller 包，新建一个 TestController 类，添加 @RequestMapping 注解

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb2fuse9uj30hp05idgb.jpg)

@RestController = @Controller + @ResponseBody（可以返回字符串或Json对象）

### 2. 增删改查

```
GET：发送查询
POST：新增
PUT：修改
DELETE：删除
```

如果只是简单的额使用 @RequestMapping 注解，表示这个接口支持所有的请求方式，也就是说 GET/POST/PUT、DELETE 都行，如果约定只能 GET ，那使用 GETMapping。

### 3. 错误码 405

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb2ktp6taj318s0emwgh.jpg)

表示：请求接口的模式不支持，可能只支持 GET，但使用了 POST

### 4. 错误码 404

表示：接口不存在

## 四、使用 http client 测试接口

### 1. 背景

平时开发如果经常要切到浏览器，会影响开发效率。

另外一点浏览器是没法执行 post 请求的。

用postman也会有一个窗口切换的问题

### 2. 使用方式

创建一个文件，以.http结尾

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb2trilgoj30s10humz8.jpg)

可以使用 Live Template 快速构建测试脚本。

### 3. 使用 HTTP client还可以构造测试用例

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb2wswhz2j310u0iz0w8.jpg)

## 五、SpringBoot的配置文件介绍

### 1. SpringBoot 至少能自动识别四种配置文件：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb308y45nj30s70c6wfm.jpg)
- config/application.properties
- config/application.yml
- application.properties
- application.yml

### 2. bootstrap.properties 文件可以实现动态配置

线上可以实时修改实时生效的配置，一般可配上 nacos 使用

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb371puhij30e004p3yy.jpg)

### 3. yaml和property格式互转

https://toyaml.com/index.html

### 4. 自定义配置项

在`application.properties`中新增：

`test.hello=Hello`

在读取的地方加上 : 

`@Value("${test.hello}")`注解

还可以配置默认值：

`@Value("${test.hello:TEST}")`注解

## 五、集成热部署

之前每次改完代码都要重启应用，有了热部署 代码改完，即时生效。

有三个步骤：

### 1. 添加依赖

pom.xml 添加：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3j1act5j311u0k2afj.jpg)

不需要加版本号

### 2. 勾选compile 静态自动编译

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3kdp90aj312o0ljdl1.jpg)

### 3. 添加 action 动态自动编译

按两次 shift 键，输入 registry ，勾选：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3lfptnyj31150lc423.jpg)

点击 Build ，就不用等待热部署了：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3mt8nmwj30di03ut8m.jpg)

## 相关高频题目

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3o06655j30mc0enach.jpg)

## 总结笔记

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3p2665bj30u01vzgrj.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb3pdxepnj31js0u0788.jpg)


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)