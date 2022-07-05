---
title: Nodejs 学习笔记
date: 2021-07-11 22:14
categories: 大前端
tags: [Nodejs]
---

# Nodejs 学习笔记

## 一、Nodejs介绍

（一）基本介绍

Node.js 是一个 JavaScript 运行环境（runtime），它让 JavaScript 可以开发后端程序，几乎能实现其它后端语言的所有功能。

Nodejs最擅长的是处理高并发：在 Java、PHP 或者 .net 等服务器端语言中，会为每一个客户端连接创建一个新的线程。而每个线程需要耗费大约 2MB 内存。也就是说，理论上一个 8GB 内存的服务器可以同时连接的最大用户数为 4000 个左右。

要让 Web 应用程序支持更多的用户，就需要增加服务器的数量，而 Web 应用程序的硬件成本就上升了。

Node.js 不为每个客户连接创建一个新的线程，而仅仅使用一个线程。当有用户连接，就触发一个内部事件，通过非阻塞 I/O 、事件驱动机制，让 Node.js 程序宏观上也是并行的。

使用 Node.js，一个 8GB内存 的服务器，可以同时处理超过4万用户的连接。

（二）开发工具

推荐使用 VSCode

安装好 Node Snippets 插件

## 二、HTTP模块、URL模块、supervisor工具

### （一）HTTP模块

如果使用 PHP 来编写后端diamante，需要 Apache 或者 Nginx 的 HTTP 服务器，来处理客户端的请求响应。不过对 Node.js 来说，概念完全不一样。使用 Node.js 时，我们不仅仅在实现一个应用，同时还实现了整个 HTTP 服务器。

Tips : ctrl + c 终止服务器

```
// 表示引入 http 模块
var http = require('http');

/*
    request ： 获取 url 传过来的信息
    response ： 给浏览器响应信息
*/
http.createServer(function (request, response) {
  // 设置响应头
  response.writeHead(200, {'Content-Type': 'text/plain'});
  // 表示给我们的页面上面输出的一句话并且结束响应
  response.end('Hello World');
}).listen(8081); // 端口

console.log('Server running at http://127.0.0.1:8081/');
```

### （二）URL模块

url 模块也是 nodejs 内置模块。

![](https://tva1.sinaimg.cn/large/008i3skNgy1grst5mcyhuj30me0lm76a.jpg)

通过 parse 并设置 true， 会将参数转成对象，从对象中取值。

```
const url = require('url');

var api = 'http://www.itying.com?name=zhangsan&age=20'

// console.log(url.parse(api,true));

var getValue = url.parse(api,true).query;

// console.log(getValue)

console.log(getValue.name)
```

### （三）Nodejs自启动工具 supervisor

supervisor 会不停的 watch 应用下面的文件，发现有文件被修改，会立刻看到变更后的记过，无需重新启动 nodejs

1. 安装 supervisor

npm install -g supervisor

2. 使用 supervisor 代替 node 命令启动应用

supervisor app.js

## 二、CommonJs 和 Nodejs模块

用 CommonJS API 编写出的应用，不仅可以利用 JavaScrip 开发客户端应用，而且还可以编写以下应用：

1. 服务器端 JavaScript 应用程序
2. 命令行工具
3. 桌面图形界面应用程序

CommonJS就是模块化的标准，nodejs就是 CommonJS（模块化）的实现。

### （一）模块

1. 核心模块：Node提供的模块

2. 文件模块：用户编写的模块

### (二）CommonJS（Nodejs）中自定义模块的规定

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvmg09akzj30s20ho7cw.jpg)

### （三）使用demo

tools.js

```
function formatApi(api) {
    return "http://www.baidu.com" + api;
}

exports.formatApi = formatApi;
```

如果是暴露一整个对象的所有方法，可以通过 ：

module.exports.xxx = obj;

如果是想一个一个暴露单个方法，可以通过exports 进行暴露：

```
exports.xxx = function() {
	console.log('hello world')
};
```

request.js

```
exports.get = function() {
	console.log('hello world')
};

exports.post = function() {
	console.log('hello world')
};
```

使用场景：

```
const request = require('./request.js');

request.get();
```

### (四)如何require的时候不写目录

var db = require('db'); 

// 自己创建的会导入失败，因为 Nodejs 会默认查找 node_modules 对应模块里面的 index.js

可以配置 package 实现可以直接 require 文件

npm init --yes

package.json 是配置文件

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvn7bzyvhj30qy0h7dp4.jpg)

## 三、Nodejs中的包、npm、第三方模块、package.json以及cnpm

### （一）包与NPM

1. 包

Nodejs 中除了它与自己提供的核心模块外，我们还可以自定义模块，也可以使用第三方的模块。

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvnaq3sk7j30lo0bz3zr.jpg)

2. 完全符合 CommonJs规范的包目录一般包含如下这些文件：

（1）package.json：包描述文件

（2）bin：用于存放可执行二进制文件的目录

（3）lib：用于存放 JavaScript 代码的目录

（4）doc：用于存放文档的目录

在 NodeJs 中通过 NPM 命令来下载第三方的模块（包）。

3. npm

npm是世界上最大的开放源代码的生态系统，可以通过npm下载各种各样的包和工具。

允许用户将自己编写的包或命令行程序上传到 NPM 服务器供别人使用。

（1）安装模块

```
示例：安装 md5 模块的步骤：

（1）在 https://www.npmjs.com/ 中找到相应的模块

（2）在自己相应的目录里安装模块 npm install md5 --save

添加了 --save ，会在 package.json 中展示相应的依赖，然后可以通过 npm i 或者 cnpm i 就会把项目所需要的依赖一个一个安装上。

（3）引入使用
```

（2）卸载模块

npm uninstall xxxx

（3）查看项目里的包

npm list

（4）查看包的信息

npm info md5

（4）指定版本安装

npm install xxx@2.1.4

Node-Media-Server ： 通过 Node-Js 搭建流媒体服务器

（5） npm i 

如果删掉 node_modules，可以通过此命令找到 package.json 找到对应的所有包信息

### （二）package.json

package.json 定义了项目的各种模块，以及各种配置信息和依赖。

"md5":"^2.2.1" 

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvob4zgqmj30o40g2dlh.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvobtbwzij30h009w0w7.jpg)

如果要指定就用某一个版本，把 ^ ~ * 标识符去掉就好了

* > ^ > ~

### （三）淘宝镜像

淘宝 NPM 镜像是一个完成的 npmjs.org 镜像，同步频率为 10分钟 一次，解决 npm install 速度慢的问题。

## 四、Nodejs中的fs模块

fs 模块是内置模块，只要用于文件操作

```
1. fs.stat 检测是文件还是目录
2. fs.mkdir 创建目录
3. fs.writeFile 创建并写入文件
4. fs.appendFile 追加文件
5. fs.readFile 读取文件
6. fs.readdir 读取目录下的所有文件名
7. fs.rename 重命名
8. fs.rmdir 删除目录
9. fs.unlink 删除文件
```

### (一)实际用例

```
const fs = require('fs');

// 1. fs.stat 检查是文件还是目录

fs.stat('./html',(err,data) => {
    if(err) {
        console.log(err);
        return
    }
    console.log('是文件:' + data.isFile())
    console.log('是文件:' + data.isDirectory())
}) 
```

```
// 2.fs.mkdir 创建目录，如果已存在目录，则失败
fs.mkdir('./css',(err)=>{
    if(err) {
        console.log(err);
        return;
    }
    console.log('创建成功');
})
```

```
// 3. fs.writeFile 创建写入文件
// 如果文件已经存在，则会覆盖文件
fs.writeFile('./html/index.html','你好nodejs',(err)=> {
    if(err) {
        console.log(err);
        return;
    }
    console.log('创建写入文件成功');
})
```

```
// 4. fs.appendFile 追加文件
// 如果文件已存在，则会在文件尾部追加内容
fs.appendFile('./css/base.css','你好nodejs',(err)=> {
    if(err) {
        console.log(err);
        return;
    }
    console.log('追加文件成功');
})
```

```
// 5. fs.readFile 读取文件
// 读取到的是十六进制的buffer数据，要转换成string类型
fs.readFile('./html/index.html',(err,data)=>{
    if(err) {
        console.log(err);
        return;
    }
    console.log(data.toString());
})
```

```
// 6. fs.readdir 读取目录下的所有文件名
fs.readdir('./html',(err,data)=>{
    if(err) {
        console.log(err);
        return;
    }
    console.log(data.toString());
})
```

```
// 7. fs.rename 重命名
// 功能1：重命名
// 功能2：移动文件
fs.rename('./css/base.css','./css/index.css',(err)=>{
    if(err) {
        console.log(err);
        return;
    }
    console.log('重命名成功');
})
```

```
// 8. fs.rmdir/unlink 删除目录
// 当目录里有文件时，不能直接删除目录，要通过 fs.unlink 先把文件删除，再使用 rmdir 删除目录
fs.unlink('./html/index.html',(err) =>{
    if(err) {
        console.log(err);
        return;
    }
    console.log('删除文件成功');
})
```

### （二）fs模块的使用

#### 实例1：

判断服务器上面有没有upload目录，如果没有创建这个目录，如果有的话不做操作（图片上传）

```
const fs = require('fs');

// 实例1：判断服务器上面有没有upload目录，如果没有创建这个目录，如果有的话不做操作（图片上传）

var path = './upload';
fs.stat(path,(err,data) => {
    if(err) {
        // 执行创建目录
        mkdir(path);
    }
    if(!data.isDirectory()){
        // 首先删除文件，再去执行创建目录
        fs.unlink(path,(err)=>{
            if(!err) {
                mkdir(path);
            }else{
                console.log('请检测传入的数据是否正确');
            }
        })
    }
})

function mkdir(dir) {
    fs.mkdir(dir,(err)=> {
        if(err) {
            console.log(err);
            return
        }
    });
}
```

#### 实例2：

介绍 mkdirp 这个模块，这个创建层级目录 ./xxx/aaa/bbb

1. https://www.npmjs.com/package/mkdirp

2. npm i mkdirp --save

3. var mkdirp = require('mkdirp');

4. 看文档使用

#### 实例3：

wwwroot文件夹下面有 images css js 以及 index.html,找出 wwwroot 目录下面的所有目录，并放到一个数组中

fs 所有的方法其实是一个异步方法，循环是做不到的。

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvq3lrbapj30lj0gbagd.jpg)

解决方式：

1. 改造for循环，用递归实现

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvq8f8hjej30or0jpgt0.jpg)

2. nodejs里面的新特性 async await

## 五、async await 的使用，以及使用 async await 处理异步

### （一）ES6 常见语法的使用 

1. let const

let 是一个块作用域

let 和 var是一样的用来定义变量的

```
if(true) {
    var a = 123;
}
console.log(a);   // 是会打印出 123 的
```

```
if(true) {
    let a = 123;
}
console.log(a);  // 会报错没有定义a变量
```

const 就是不可改变

2. 箭头函数

3. 对象、属性的简写

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvqk13tjaj30ib0gdq72.jpg)

4. 模板字符串

```
var name = '张三';
var age = 20;
console.log(`${name}的年龄是${age}`)
```

用的不是单引号包裹的，用的是 Tab 上的富豪 ·

5. Promise 处理异步问题

在 ES6 出来之前，使用外部获取异步方法来获取数据：

```
function  getData(callback) {
    //ajax
    setTimeout(function() {
        var name = '张三';
        callback(name);
    },1000);
}

// 外部获取异步方法里面的数据

getData(function(test){
    console.log(test)
})
```

Promise 来处理异步

```
var p = new Promise(function(resolve,reject) {
    // resolve：成功的回调
    // reject：失败的回调
    setTimeout(function() {
        var name = '张三';
        resolve(name);
    },1000);
})

p.then(function(data) {
    console.log(data);
})
```

### （二）async、await 和 promise 的使用

async：让方法变成异步

await：等待异步方法执行完成

```
async function test() {
    return '你好nodejs';
}

console.log(test());  // ==> 输出：Promise { '你好nodejs' }
```

await：必须用在 async 方法里

```
async function test() {
    return '你好nodejs';
}

async function main() {
    var data = await test(); // 获取异步方法里的数据
    console.log(data);
}

main();
```

### （三）异步方法检测目录习题

```
const fs = require('fs');

var dirArr = [];
var path = './wwwroot';

// 1. 定义一个isDir方法判断一个资源到底是目录还是文件
async function isDir(path) {
    return new Promise((resolve,reject) => {
        fs.stat(path,(error,stats)=> {
            if(error) {
                console.log(error);
                reject(error);
                return
            }
            if(stats.isDirectory()){
                resolve(true);
            }else{
                resolve(false);
            }
        })
    })
}

// 2. 获取 wwwroot 里面的所有资源，循环遍历

function main() {
    fs.readdir(path,async(err,data)=> {
        if(err) {
            console.log(err);
            return;
        }

        for(var i = 0 ; i < data.length ; i ++) {
            if(await isDir(path + '/' + data[i])) {
                dirArr.push(data[i]);
            }
        }

        console.log(dirArr);
    })
}

main();
```

## 六、Nodejs fs中的流以及管道流、复制文件、复制图片

1. fs.createReadStream 从文件流中读取数据

```
const fs = require('fs')

var readStream = fs.createReadStream('./data/input.text');

readStream.on('data',(data) => {
    // 监听读取进度，会多次回调
})

readStream.on('end',(data) => {
    // 读取完毕
})

readStream.on('error',(data) => {
    // 读取失败
})

```

2. fs.createWriteStream 向文件中写入流数据

```
const fs = require('fs')

var str = '';

for(var i = 0 ; i < 100; i ++) {
    str += 'test';
}

var writeStream = fs.createWriteStream('./data/output.text');

writeStream.write(str);

writeStream.on('finish',()=>{
    console.log('write suc')
})
```

3. 管道流 ： 将数据从一个文件复制到另外一个文件，应用场景：大文件、图片

![](https://tva1.sinaimg.cn/large/008i3skNgy1grvs3zwylcj30ki0b1799.jpg)

## 七、利用 HTTP模块、Url模块、Path模块、fs模块创建一个静态 WEB 服务器

预期实现的目标：

1. 可以让我们访问web服务器上面的网站

```
var http = require('http');
var fs = require('fs');
const common = require('./common');
const { runInNewContext } = require('vm');
const path = require('path');

http.createServer(function (req, res) {

    // 1. 获取地址
    let pathName = req.url;
    pathName = (pathName == '/') ? '/index.html' : pathName; // 重定向
    let extname = path.extname(pathName); // 可以获取后缀名
    console.log(pathName);

    // 2. 通过fs模块读取文件
    if (pathName != '/favicon.ico') {
        console.log('begin read File');
        fs.readFile('./' + pathName,(err,data) => {
            if (err) {
                console.log('read File fail');
                res.writeHead(404, {'Content-Type': 'text/html;charset="utf-8"'});
                res.end('页面不存在');
            } else {
                console.log('read File suc');
                let mime = common.getMime(extname);
                res.writeHead(200, {'Content-Type': ''+mime+';charset="utf-8"'});
                res.end(data);
            }
        })
    }
}).listen(8081);

console.log('Server running at http://127.0.0.1:8081/');
```

## 八、Express 介绍

Express 是一个基于 Node.js 平台，快速、开放、极简的 web 开发框架，它提供一系列强大的特性，帮助你创建各种 Web 和 移动设备应用。

### （一）基本请求类型

```
const express = require('express');
const app = express(); // 实例化 express

// 配置路由:get\post\put\delete
app.get('/',(req,res)=>{
    res.send('你好 express');
})

app.get('/admin/user',(req,res)=>{
    res.send('配置二级目录');
})

// post是无法在浏览器里直接请求的，可以使用 Postman 软件模拟请求
app.post('/doLogin',(req,res)=>{
    res.send('执行登录');
})

// put 表示修改数据，也是一种请求类型
app.put('/editUser',(req,res)=>{
    res.send('修改用户');
})

// delete 删除数据
app.delete('/deleteUser',(req,res)=>{
    res.send('删除数据');
})

app.listen(3000); // 监听3000端口
```

### (二)动态路由与动态路由配置顺序

```
const express = require('express');
const app = express(); // 实例化 express

// 动态路由，配置动态路由是有顺序的，如果想单独配置 /article/add，要在'/article/:id'动态路由前配置好，不然会被动态路由捕获
app.get('/article/:id',(req,res)=>{
    var id = req.params['id'];
    res.send('动态路由'+id);
})

app.listen(3000); // 监听3000端口
```

### （三）获取get请求中的参数

```
const express = require('express');
const app = express(); // 实例化 express

// get 传值 
app.get('/product',(req,res)=>{
    let query = req.query;
    console.log(query);
    res.send('product-' + query.id);
})

app.listen(3000); // 监听3000端口
```

### 九、Express Ejs 静态文件托管

####（一）绑定数据 

<%=xxx%> 绑定对象和属性

####（二）解析 html

<%-xxx%> 可以解析 xxx 对应的 html 代码，

比如 xxx 可以为 ：

```
const express = require('express');
const app = express(); // 实例化 express
 // 配置ejs模板引擎（因为express默认集成ejs）
app.set('view engine','ejs');

app.get('/news',(req,res)=> {
    var userInfo = {
        username:'张三',
        age:20
    }

    let article = "<h3>我是一个h3</h3>"
    
    res.render('news',{
        userInfo:userInfo,
        article:article
    });
})

app.listen(3000); // 监听3000端口，建议写成3000以上，3000以下的端口会被计算机软件占用
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h2>我是一个ejs模板引擎</h2>
    <p><%=userInfo.username%></p> 
    <p><%-article%></p> 
</body>
</html>
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1grx8llzrs7j30hg09ogm0.jpg)

####（三）条件判断

```
    <%if(score>=60){%>
        <p>及格</p>
    <%}else{%>
        <p>不及格</p>
    <%}%>
```

####（四）循环遍历

![](https://tva1.sinaimg.cn/large/008i3skNgy1grx8som335j30gc073jt5.jpg)

####（五）在ejs模板里引入其他模板

![](https://tva1.sinaimg.cn/large/008i3skNgy1grx8varozgj30xn0f2k05.jpg)

####（六）指定模板位置，默认模板位置在 views

app.set('views',__dirname + '/views');

####（七）Ejs 后缀修改为 html

![](https://tva1.sinaimg.cn/large/008i3skNgy1grx92p3qv9j30gs0diaeb.jpg)

#### （八）中间件

通俗地讲：中间件就是匹配路由之前或者匹配路由完成做的一系列的操作，中间件中如果想往下匹配的话，那么需要些 next()

![](https://tva1.sinaimg.cn/large/008i3skNgy1grx97tav8xj30mx03caay.jpg)

Express 应用可使用如下几种中间件：

1. 应用级中间件（用于权限判断）

![](https://tva1.sinaimg.cn/large/008i3skNgy1grx997h20xj30lt08qad6.jpg)

实现在匹配 get/post 之前，打印下当前日志：

```
// 应用级中间件
app.use((req,res,next)=>{
    console.log(new Date());
    next();//如果不写这个next,匹配到这里就结束了
})
```

2. 路由级中间件（用得比较少）

通过添加 next，可以让路由匹配到 /article/add 之后，还可以继续匹配  /article/:id。

```
app.get('/article/add',(req,res,next)=>{
    console.log('路由级中间件');
    next();
})

app.get('/article/:id',(req,res)=>{
    var id = req.params['id'];
    res.send('动态路由'+id);
})
```

3. 错误处理中间件

针对匹配不到的路由，返回状态码 404，相当于兜底逻辑

```
app.use((req,res,next)=>{
    res.status(404).send("404");
})
```

4. 内置中间件(匹配路由之前，先在内置中间件查找相应的文件)

app.use(express.static('static'))

5. 第三方中间件

（1）body-parser 获取post传过来的数据

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4rp2fajsj30kw0cmdjh.jpg)

### 十、在Express中使用 Cookie

Cookie 是存储于访问者的计算机中的变量，可以让我们用同一个浏览器访问同一个域名的时候共享数据。

HTTP 是无状态协议，简单地说：当你浏览了一个页面，然后转到同一个网站的另一个页面，服务器无法认识到这是同一个浏览器在访问同一个网站。每一次访问都是没有任何关系的。

Cookie 是一个简单到爆的想法：当访问一个页面的时候，服务器在下行HTTP报文中，命令浏览器存储一个字符串；浏览器再访问同一个域的时候，将把这个字符串写到到上行HTTP请求中。

第一次访问一个服务器，不可能携带Cookie，必须是服务器得到这次的请求，在下行响应报文中，携带Cookie信息，此后每一次浏览器往这个服务器发出的请求，都会携带这个cookie。

#### （一）cookie-parser 的使用

```
const express = require('express');
const app = express(); // 实例化 express
const cookieParser = require('cookie-parser');

// 配置路由:get\post\put\delete
app.get('/',(req,res)=>{
    // 设置cookie
    res.cookie('key_mock','value_mock',{maxAge:1000*60*60});// 设置cookie过期时间 maxAge
    res.send('你好 express');
})

app.get('/admin/user',(req,res)=>{
    // 获取cookie
    var key_mock = req.cookies.key_mock;
    console.log(key_mock);
    res.send('配置二级目录');
})
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4scf8h7uj31sa0k2490.jpg)

后期使用 cookie 需要对 cookie 进行加密

Cookie 配置的参数：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4se2nm6uj30ij09qn1c.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4vybx62wj30sa0gcws4.jpg)

- maxAge ： 过期时间
- expires ： 设置具体过期时期（用得比较少）
- path：设置cookie访问的目录（也即哪些路由可以访问cookie）
- httpOnly：true，只能在后端设置cookie，false表示前端也可以设置cookie
- secure：true表示在HTTP无效，只有HTTPS才是有效的
- signed：表示是否签名 cookie，设为true会对这个cookie签名，这样就需要用 res.signedCookies 而不是 res.cookies 访问它。被篡改的签名cookie会被服务器拒绝，并且cookie值会重置为它的原始值。

多个域名共享 cookie ： 同个域的二级域名共享 cookie 

- aaa.itying.com  
- bbb.itying.com

```
res.cookie('key_mock','value_mock',{maxAge:1000*60*60,domain:'.itying.com'});
```

#### （二）cookie的加密

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4w8bruz4j30t70fl0yn.jpg)

cookie 的加密

1. 配置中间件的时候需要传入加密的秘钥

app.use(cookieParser('itying'))

2. 设置 cookie
res.cookie('key_mock','value_mock',{maxAge:1000*60*60,signed:true});

3. 获取cookie
let username = res.signedCookies.username

### 十一、Session 的基本使用

Session 是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而session保存在服务器上。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4wa04todj30td0aqthr.jpg)

#### （一）Session的工作流程

>> 浏览器访问服务器并发送第一次请求

>> 服务器端会创建一个session对象，生成一个类似于 key、value 的键值对

>> 然后将key（cookie）返回到浏览器（客户）端

>> 浏览器下次再访问时，携带key（cookie）请求服务器，找到对应的session（value）

Tips：一个get请求只能写一个 req.send ，写两个send 会导致失败

```
const express = require('express')
const session = require('express-session')
const app = express()
const port = 3001
// 配置session的中间件
app.use(session({
    secret: 'keyboard cat', // 服务器端生成 session 的签名
    name: 'itying', //修改session对应的cookie名称（即key）
    resave: false, // 强制保存 session 即使它并没有变化
    saveUninitialized: true, // 强制将未初始化的 session 存储
    cookie: {
        maxAge: 1000 * 60,
        secure: false // true表示只有https协议才能访问 cookie
    },
    rolling: true // 在每次请求时强行设置 cookie，这将重置 cookie 过期时间（默认:false）
}))

app.get('/', (req, res) => {
    // 获取session
    if (req.session.username) {
        res.send(req.session.username + '-已登录')
        return;
    }
    res.send('没有登录!')
})

app.get('/login', (req, res) => {
    // 设置session
    req.session.username = '张三'
    res.send('已登录')
})

app.listen(port, () => console.log(`Example app listening on port port!`))
```

Tips2: VSCode 代码格式化 快捷键 On Mac 　　Shift + Option + F

#### (二)销毁session

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4x3zglkqj30l60cun1w.jpg)

```
app.get('/loginOut',(req,res)=>{
    // 1. 设置 Session 的过期时间为0（它会把所有的session都销毁）
    req.session.cookie.maxAge = 0;

    // 2. 销毁指定session
    req.session.username = '';

    // 3. 使用 destroy 销毁 session
    req.session.destroy()

    res.send('退出登录!')
})
```

### 十二、分布式架构配置 Session 

在北京服务器设置了 session，然后负载均衡，Nginx 服务器可能会在上海的服务器里取获取 session。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4y92s0gxj30q80bfmyt.jpg)

所以这时需要把 session 放到数据库中，比如使用 mango DB。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs4y9vlehoj30wp0bcdhy.jpg)

#### （一）connect-mongo 

后台通过数据库解决数据不一致的问题，可选的数据库：

- mongoDB
- mysql
- redis

```
/*
session保存在数据库里面

1. 配置 express-session

2. 安装 connect-mongo

3. 引入mongo
const MongoStore = require('connect-mongo')(session);

4. 配置中间件
app.use(session({
    secret: 'keyboard cat', // 服务器端生成 session 的签名
    name: 'itying', //修改session对应的cookie名称（即key）
    resave: false, // 强制保存 session 即使它并没有变化
    saveUninitialized: true, // 强制将未初始化的 session 存储
    cookie: {
        maxAge: 1000 * 60,
        secure: false // true表示只有https协议才能访问 cookie
    },
    rolling: true, // 在每次请求时强行设置 cookie，这将重置 cookie 过期时间（默认:false）
    store: new MongoStore({
        url: 'mongodb://127.0.0.1:27017/shop',
        touchAfter:24 * 3600 // 不管发出了多少请求，在24小时之更新一次session，除非你改变了session
    })
}))
*/
```

### 十三、Express 路由模块化

Express 中允许我们通过 express.Router 创建模块化的、可挂载的路由处理程序。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs50c3rolvj30ti0lcna6.jpg)

把功能相似的路由进行模块化处理（有点类似JavaScript使用 exports 暴露方法和属性，只是express 把提供的 get方法 进行了 exports的处理）

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs50gx1r0rj30q20gjth3.jpg)

挂载 login模块：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gs50hijxkhj30sf0hsdq1.jpg)


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)