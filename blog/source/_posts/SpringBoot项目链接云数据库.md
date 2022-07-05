---
title: SpringBoot项目链接云数据库
date: 2022-01-03 01:46
tags: [SpringBoot,idea,云数据库]
categories: [后台]
---

# SpringBoot项目链接云数据库

章节来自：后端架构完善与接口开发

## 一、数据库准备

### 1. 本地数据库准备

#### （1）安装阶段

使用 `brew install mysql`安装，发现安装失败：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb50rlmfzj310a03yaal.jpg)

执行 `brew update`之后再执行`brew install mysql`就可以执行成功了。

配置可以参考这篇文章：https://juejin.cn/post/6844903831298375693#heading-1

#### （2）navicat 连接到本地 mysql

创建数据库，字符集 utf8mb3 

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb5q0di06j311k0rk0uj.jpg)

#### （3）单独创建一个访问数据库的用户，不能一直用 root用户，root权限太大了

localhost表示只有本地能用这个用户登录，用%表示可以在别的机器上用这个用户登录

添加权限：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb5y3ujewj30t40jpjsp.jpg)

#### （4）创建一个连接，专门用来连接 wiki用户

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxb60r7x9wj307u05rdfu.jpg)

### 2. 阿里云数据库准备

#### （1）查看实例列表

点击 云数据库RDS网站：https://rdsnext.console.aliyun.com/rdsList/cn-shenzhen/basic

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxyidymrv9j30og08rgm3.jpg)

点击`管理`：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxyiej5xflj30g804g74b.jpg)

#### （2）选择数据库管理

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxyifpozxoj30m20ast93.jpg)

所有的项目都可以用同一个数据库。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxbgqsv6sqj30ff0jdgm3.jpg)

#### （3）创建数据库之前创建一个专有的帐号

点击 网站：https://rdsnext.console.aliyun.com/detail/rm-wz95hzt16zd6a59dh/account/list?region=cn-shenzhen

选择 `创建帐号`

#### （4）重新创建数据库，绑定新创建的帐号

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxyilzb9taj30gw08v74g.jpg)

#### （5）登录数据库

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxyimskr3cj30l009a0sz.jpg)

登录DMS数据管理服务网站后，如果当前所选帐号不是我们创建的，那么我们可以选择`切换账户`：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxzsr5lrqfj30d904lq2x.jpg)

接着就可以发现该账户下绑定的数据库列表：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxzsstwpc1j307707e74e.jpg)

也就是说，阿里云数据库中：帐号和数据库是绑定的。

#### （6）使用idea连接云数据库

在 阿里云 工作台选择 数据库连接，即可看到 内网地址/外网地址：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h008lpkaubj20u00xg77j.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxztloz5h0j30x50u0q5z.jpg)

操作完后，可以发现我们创建的数据库，以及对应的sql控制台：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxztnjp1alj30n5079js1.jpg)

#### （7）使用sql语句为数据库添加表

在`console`指令中输入如下命令：

```
create table `test` (
    `id` bigint not null comment 'id',
    `name` varchar(50) comment '名称',
    primary key (`id`)
)  engine = innodb default charset=utf8mb4 comment='测试';
```

Tips：注意区分关键词用符号 ``，字符串用符号 ''

可以发现，表格创建成功：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxztppnkisj30t70fxdhf.jpg)

#### （8）执行select语句

```
select  * from test
```

得到执行的结果：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxztrpn3okj30kn0d7wf6.jpg)

因为我们只是创建了表，但没有给表添加数据，所以得到的结果是一张空表。

目前我们就得到了一个mysql客户端了。

#### (9) 本地新建数据并提交云端

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxztu7bpywj30pc04ajrk.jpg)

提交后，我们到我们的阿里云数据库查看，可以发现数据已经提到了云端：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxzty63sryj30pg0eh3zo.jpg)



------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)