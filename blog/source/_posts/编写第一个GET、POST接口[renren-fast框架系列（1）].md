---
title: 编写第一个GET、POST接口[renren-fast框架系列（1）]
date: 2022-03-13 00:24
tags: [Postman,Post]
categories: [后台]
---

# 编写第一个GET、POST接口[renren-fast框架系列（1）]

配置好 renren-fast 脚手架，学习完 Spring MVC 架构后，我需要具体调试 renren-fast 的接口，比如要新增某个接口。

## 什么是前后端分离

运行 renren-fast 项目时，我们访问 http://localhost:8080/renren-fast/ 的结果：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07a28q6knj20b404ct8p.jpg)

可以看到，接口给出了相应的回应，状态码 401 Unauthorized 代表客户端错误，指的是由于缺乏目标资源要求的身份验证凭证，发送的请求未得到满足。

运行 renren-fast-vue 项目时，我们访问 http://localhost:8001/#/login ：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07a4qnsr7j21100mztac.jpg)

接着使用Chrome自带的网络工具：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07h9rbsh7j20k90h1q4a.jpg)

点击 Headers 可以查看 Request URL：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07hb9tn9oj20rv0bugne.jpg)

以此，我们确认访问 后台接口为：http://localhost:8080/renren-fast/captcha.jpg?uuid=11e75e91-f584-4e38-804b-d6776535eed9

同时还可以看到如下获取页面信息的 headers：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07hcj2wytj20kc04s0tb.jpg)

请求的访问接口是：http://localhost:8001/static/config/index.js

至此，我们便搞清楚了 renren-fast 前后端分离的业务是在说什么，即：后端逻辑使用 renren-fast 实现，前端请求获取的页面数据，使用 renren-fast-vue 实现。

## 写一个无需鉴权的GET接口

接下来我们基于 renren-fast 框架仿照 "/captcha.jpg" 写第一个接口， "/captcha.jpg"接口代码：

```
	/**
	 * 验证码
	 */
	@GetMapping("captcha.jpg")
	public void captcha(HttpServletResponse response, String uuid)throws IOException {
		response.setHeader("Cache-Control", "no-store, no-cache");
		response.setContentType("image/jpeg");

		//获取图片验证码
		BufferedImage image = sysCaptchaService.getCaptcha(uuid);

		ServletOutputStream out = response.getOutputStream();
		ImageIO.write(image, "jpg", out);
		IOUtils.closeQuietly(out);
	}
```

我们写一个 /testInterface 接口，在没写接口之前，我们访问： http://localhost:8080/renren-fast/testInterface ，返回如下样式：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07i3ds89lj208b01ewea.jpg)

我们编写接口：

```
	/**
	 * 测试接口
	 */
	@GetMapping("/testInterface")
	public String testInterface() {
		return "hello";
	}
```

部署完之后访问： http://localhost:8080/renren-fast/testInterface ，仍旧是展示：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07i3ds89lj208b01ewea.jpg)

那问题来了，我们这个接口部署的有问题吗？ 显然是没有问题的，只是一个简单示例，接着我们在代码中搜索 "{"msg":"![invalid token]","code":401}" ，可以发现有这么一句：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07i6ed1mwj20pp0e40ua.jpg)

这意味着即使我们部署了接口，如果token不存在，也依旧会返回 401，但我们这个接口不需要 token 鉴权，所以要配置无需鉴权的逻辑。

我们从哪里可以知道怎么配置如需鉴权呢？ 还记得我们上面发现的一个无需鉴权的接口吗？ 

http://localhost:8080/renren-fast/captcha.jpg?uuid=11e75e91-f584-4e38-804b-d6776535eed9

/captcha.jpg 这个接口无需登录，只要任意修改uuid就能获取验证码，所以我们全局搜索 /captcha.jpg，可以发现有3处地方：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07ichmbctj209u058mx8.jpg)

一处是配置 filterMap，也就是无需鉴权的接口，我们在同样的地方加上我们自己的 testInterface 接口：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07idjjr6fj20kk0df3zv.jpg)

另一处是配置接口参数、描述、回包数据等，我们也简单配置一下：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07iekl4m5j20kw0gygmn.jpg)

配置完毕后重新部署，重新访问 http://localhost:8080/renren-fast/testInterface  ，可以发现页面展示：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07if59wu8j20c004fmx6.jpg)

接口部署成功！

## 调试一个需要鉴权的GET接口

GET 接口可以简单理解为从服务器获取数据，只是读数据；而POST接口允许通过请求让服务器做一些事情：比如写数据、删数据等等。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07irquivvj20p80bf0ua.jpg)

当我们登录时，是向 http://localhost:8080/renren-fast/sys/login 发送 POST 请求，我们来使用 Postman 模拟发送 post请求，

首先，我们要获取 验证码，我们请求： http://localhost:8080/renren-fast/captcha.jpg?uuid=11e75e93 ，获取验证码：b676b

接着我们查看 /sys/login 接口代码得知：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07ix2itkcj20q50gxjsn.jpg)

登录接口所必须的参数有：
- uuid
- captcha
- username
- password

使用Postman发送Post请求成功：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07jzuhucqj21ab0u0q6m.jpg)

我们获取 token 为 5d68f7779618adba8a1fb37df4bbe8e6 ，那是不是意味着我们可以通过 token 访问那些有权限的接口了呢？我们试试：

我们使用 Postman 发送 GET 请求： http://localhost:8080/renren-fast/sys/menu/nav?t=1647101933260&token=5d68f7779618adba8a1fb37df4bbe8e6

果然回包成功了：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h07k5ub4pgj213w0u00vv.jpg)

# 小结

通过本篇文章的学习，我们明确了如下的内容：
- renren-fast 前后端分离的设计理念
- 学会配置一个无需鉴权的简单接口
- 通过postman调试了鉴权post逻辑和get逻辑

至此我们可以自由编写 鉴权接口 和 非鉴权接口。

-----------------
文章首发：[问我社区](http://www.wenwoha.com/)