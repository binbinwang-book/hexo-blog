---
title: SpringBoot项目搭建流程
date: 2022-03-01 12:05
tags: [SpringBoot]
categories: [后台]
---


# SpringBoot项目搭建流程
[TOC]

文章首发：http://www.wenwoha.com/?cat_id=5

## 一、搭建SpringBoot项目

### 两种方法创建SpringBoot项目

#### 方法一：通过官网获取框架

进入 https://start.spring.io/ 网站，打成一个jar包，依赖只添加一个 Spring Web。

下载完成之后解压通过 idea 打开即可。

#### 方法二：通过idea获取配置

idea -> File -> New -> Project -> Spring Initializr

### SpringBoot项目结构

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzskqnruf1j21260u00v8.jpg)

### 启动SpringBoot项目

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzskrq0l1ij20u00y342t.jpg)

## 二、项目初始配置

### 编码配置

所以GDK配置修改为 UTF-8 :

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzskzjhapyj21ey0u0af1.jpg)

配置完毕后，项目右下角会变成 UTF-8

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzsl04w5ryj20bs054t8q.jpg)

### 配置git

启用idea进行git配置：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzsl2b1yz7j211j0u0n13.jpg)  

可以发现文件颜色发生变化，红色表示还没有交给git进行管理：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzsl43n460j20xa0pwtbq.jpg)

我们需要把红点add进git来管理，选中项目，然后点击项目或某文件，点击 git -> Commit Directory

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzsldccw79j20xs0u0jw1.jpg)

勾选需要提交的文件，然后点击commit提交，

当我们需要push时候，点击 Git -> Push（如果没有配置仓库的，在页面上配置一下即可，github支持快捷登录），弹出页面：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzsle8cjrpj219e0t476n.jpg)

点击push，push成功：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzslendjrbj20ks058dfv.jpg)


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)