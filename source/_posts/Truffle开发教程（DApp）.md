---
title: Truffle开发教程（DApp）
date: 2021-11-14 23:06
tags: [元宇宙]
categories: [区块链]
---

# Truffle开发教程（DApp）

作为一名iOS工程师，除了对App开发感兴趣，最近比较火的DApp开发也吸引了我。

DApp（Decentralized Application），也即 去中心化应用，一个DApp是按如下的框架运转的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gvqhr21cgdj60ma0grwfd02.jpg)

从开发DApp的角度上出发，我们需要做的流程是：

>> 0. 选定DApp所在的区块链，比如：EOS、BSC、ETH等
>> 1. 开发智能合约
>> 2. 开发DApp的Web界面
>> 3. DApp界面和智能合约交互

我们可以发现，上面这些步骤无论开发哪个DApp都是必须的，所以有没有一个工具可以把我们开发DApp的基础框架准备好呢？

有的，就是本篇文章要介绍的 [Truffle](https://www.trufflesuite.com/boxes)。

## 一、Truffle 命令

安装 Truffle 的教程网上有很多，我就不赘述了，安装完 Truffle 后我们可以在终端输入 truffle ：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gweu0c9gldj30z40u0jwb.jpg)

我们可以看到 truffle 提供的很多指令：

```
Commands:
  build     Execute build pipeline (if configuration present)
  compile   Compile contract source files
  config    Set user-level configuration options
  console   Run a console with contract abstractions and commands available
  create    Helper to create new contracts, migrations and tests
  db        Database interface commands
  debug     Interactively debug any transaction on the blockchain
  deploy    (alias for migrate)
  develop   Open a console with a local development blockchain
  exec      Execute a JS module within this Truffle environment
  help      List all commands or provide information about a specific command
  init      Initialize new and empty Ethereum project
  install   Install a package from the Ethereum Package Registry
  migrate   Run migrations to deploy contracts
  networks  Show addresses for deployed contracts on each network
  obtain    Fetch and cache a specified compiler
  opcode    Print the compiled opcodes for a given contract
  preserve  Save data to decentralized storage platforms like IPFS and Filecoin
  publish   Publish a package to the Ethereum Package Registry
  run       Run a third-party command
  test      Run JavaScript and Solidity tests
  unbox     Download a Truffle Box, a pre-built Truffle project
  version   Show version number and exit
  watch     Watch filesystem for changes and rebuild the project automatically
```

## 二、Truffle 示例Demo和文件结构

我们新建一个空的文件夹：`TruffleDevDemo`，使用 unbox 下载 Truffle 官方提供的一个 区块链转账 Demo，执行：

>>> truffle unbox webpack

如果你在执行的时候遇到如下或者其它乱七八糟的错误，我们可以去[Truffle](https://www.trufflesuite.com/boxes)官网直接把 Demo 下载下来（本次重点是学习Truffle框架，不是解决Truffle环境问题）：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwet3xbx52j30u40f875q.jpg)

找到 Webpack 项目，然后我们点击打开查看其 github项目网站:[Webpack](https://github.com/truffle-box/webpack-box)，把代码包里的代码拷贝到我们新建的空目录 `TruffleDevDemo`即可。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwet603fpkj30al08vmxb.jpg)

接着我们可以看到项目目录中有如下的文件：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetahfx0uj311e03igm8.jpg)

我们使用 sublime 打开它，执行：

>>> subl .

打开后可以发现如下的目录结构：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetbvuz4mj30gg0wumyk.jpg)

其中最重要的几个文件和含义：

- ① web界面 index.html
- ② 智能合约 .sol
- ③ 前后端如何交互 .js脚本

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwethqcc0oj30mr0gz3zi.jpg)

## 三、执行Demo

### （一）创建开发者测试链

执行指令：

>>> truffle develop

如果执行指令报如下的错误：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetknju99j31gk0i0wjf.jpg)

需要在配置环境，在终端执行如下命令：

```
export NODE_OPTIONS=--openssl-legacy-provider
```

接着重新执行指令：

>>> truffle develop

终端上输出了如下内容：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetohcxftj311i0u0q7n.jpg)

这实际上是 Truffle 帮我们创建了一条测试链，并提供了10个测试帐号（我们开发的时候肯定不能直接基于以太坊链花真金白银去做这些事情）。

### （二）编译 sol 智能合约

在 truffle develop 环境下执行指令：

>>> compile

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwetspksa4j311i0u042t.jpg)

### （三）将编译的 sol 合约部署链上

在 truffle develop 环境下执行指令：

>>> migrate

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwettcnz44j311i0u0tbb.jpg)

接下来，我们怎么和链上的合约进行交互呢？

我们启动一个 Web 前端进行交互。

## 四、启动前端界面

我们新建一个终端，切到 `TruffleDevDemo/app` 目录下，执行指令：

>>> npm run dev

![](https://tva1.sinaimg.cn/large/008i3skNgy1gweu1tn3cnj315c0u010k.jpg)

报错：

`Error: Cannot find module 'copy-webpack-plugin'`

解决答案，执行：

>>> npm install copy-webpack-plugin -g

（发现报错是找不到哪个 module 就npm install 哪个 module 即可）

安装完成后我们继续执行指令：

>>> npm run dev

接着发现报错：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwezxl4o95j312o0h2419.jpg)

`You need to install 'webpack-dev-server' for running 'webpack serve'.`

解决方法是执行指令：

>>> npm install webpack-cli -g

解决完配置问题后，执行指令：

>>> npm run dev

发现执行成功：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwf01xnhipj312m0u0jxl.jpg)

我们打开网页：http://localhost:8080/ 发现已经执行成功了：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwf05em4r8j30hx0dd3z1.jpg)



------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)