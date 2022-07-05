---
title: 边玩边学 Solidity （笔记）
date: 2021-11-21 16:51
tags: [Solidity,区块链]
categories: [区块链]
---

# 边玩边学 Solidity （笔记）

学习网站：https://cryptozombies.io/zh

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwmvb0f2ykj313q0otdi8.jpg)

```
问题小集

1. 如何通过代码调用其它合约的接口

明确合约地址，构建合约即可调用。

2. 如何在禁止第三方修改我们合约的同时，留个后门给自己去修改。

实现修饰词 OnlyOwner。

3. 不消耗gas的函数应该声明view关键词

4. Solidity 为了实现安全的编程，做了很多保证（Owner、view、public、函数修饰符OnlyOwner）

6. 代币的本质是什么？

代币的本质是遵守固定方法的智能合约。

7. 区块链转账部分的代码需要考虑多线程问题吗？区块链如何解决多人抢票的问题？

区块链转账本质是强求安全，不追求效率。

8. 前端和区块链交互有以太坊基金会提供的 Web3.js 库，那其它终端（比如安卓、iOS）和区块链交互现在有什么库吗？

目前各个社区都在积极构建自己的 web3 库，比如：

- iOS：[web3swift](https://github.com/BANKEX/web3swift)
- Android：[web3j](https://github.com/web3j/web3j)
- php：[web3.php](https://github.com/web3p/web3.php)
- python：[web3.py](https://github.com/ethereum/web3.py)

基本所有社区的 web3 库都在以 web3.js 为模板实现各种方法，但 web3.js 本身都在飞速发展中，所以其它社区的 web3 库目前处于初步发展阶段，如果有打算加入的现在是个好时机。

```

## 1. public

创建一个数据类型为 Zombie 的结构体数组，用public修饰

```
Zombie[] public zombies;
```

## 2. 私有函数

```
    function _createZombie(string _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }
```

## 3. 只读函数，关键词：view

上面的函数实际上没有改变 Solidity 里的状态，即，它没有改变任何值或者写任何东西。

这种情况下我们可以把函数定义为 view, 意味着它只能读取数据不能更改数据:

```
    function _generateRandomDna(string _str) private view returns (uint) {
        // 这里开始
    }
```

## 4. Web3和Solidity的关系

用 Solidity 完成合约，使用 JavaScript Web3.js 调用这个合约

## 5. 映射Mapping 

映射Mapping 是Solidity 中存储有组织数据的方法，是一种KV形式的数据结构

```
mapping (uint => address) public zombieToOwner;
mapping (address => uint) ownerZombieCount;
```

## 6. 全局变量 msg.sender

指的是当前调用者（或智能合约）的 address

使用 msg.sender 很安全，因为它具有以太坊区块链的安全保障 —— 除非窃取与以太坊地址相关联的私钥，否则是没有办法修改其他人的数据的。

## 7. 断言关键词 require

```
function sayHiToVitalik(string _name) public returns (string) {
  // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
  // (敲黑板: Solidity 并不支持原生的字符串比较, 我们只能通过比较
  // 两字符串的 keccak256 哈希值来进行判断)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 如果返回 true, 运行如下语句
  return "Hi!";
}
```

## 8. 继承关键词 is

```
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

## 9. 文件导入关键词 import

```
import "./zombiefactory.sol";
```

## 10. 存储变量

- storage：永久存储在区块链中的变量
- memory：临时变量

读取区块链全局变量中的对象时，可以通过声明 storage/memory，决定是否修改对象中的数据。

## 11. 函数可见性

- public
- private
- internal：如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数（内部）
- external：只不过这些函数只能在合约之外调用 - 它们不能被合约内的其他函数调用

## 12. 与其他合约的交互（使用其它合约函数）

如果我们的合约需要和区块链上的其他的合约会话，则需先定义一个 interface (接口)。

先举一个简单的栗子。 假设在区块链上有这么一个合约：

```
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

这是个很简单的合约，您可以用它存储自己的幸运号码，并将其与您的以太坊地址关联。 这样其他人就可以通过您的地址查找您的幸运号码了。

现在假设我们有一个外部合约，使用 getNum 函数可读取其中的数据。

首先，我们定义 LuckyNumber 合约的 interface ：

```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

请注意，这个过程虽然看起来像在定义一个合约，但其实内里不同：

首先，我们只声明了要与之交互的函数 —— 在本例中为 getNum —— 在其中我们没有使用到任何其他的函数或状态变量。

其次，我们并没有使用大括号（{ 和 }）定义函数体，我们单单用分号（;）结束了函数声明。这使它看起来像一个合约框架。

编译器就是靠这些特征认出它是一个接口的。

在我们的 app 代码中使用这个接口，合约就知道其他合约的函数是怎样的，应该如何调用，以及可期待什么类型的返回值。

在下一课中，我们将真正调用其他合约的函数。目前我们只要声明一个接口，用于调用 CryptoKitties 合约就行了。

## 13. 使用合约接口

```
contract MyContract {
  address NumberInterfaceAddress = 0xab38...;
  // ^ 这是FavoriteNumber合约在以太坊上的地址
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 现在变量 `numberContract` 指向另一个合约对象

  function someFunction() public {
    // 现在我们可以调用在那个合约中声明的 `getNum`函数:
    uint num = numberContract.getNum(msg.sender);
    // ...在这儿使用 `num`变量做些什么
  }
}
```

通过这种方式，只要将您合约的可见性设置为public(公共)或external(外部)，它们就可以与以太坊区块链上的任何其他合约进行交互。

这里最神奇地是：你可以非常方便地调用其他人写的代码，但你要知道其合约所在区块链上的地址。

## 14. 处理多返回值

```
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 这样来做批量赋值:
  (a, b, c) = multipleReturns();
}

// 或者如果我们只想返回其中一个变量:
function getLastReturnValue() external {
  uint c;
  // 可以对其他字段留空:
  (,,c) = multipleReturns();
}
```

## 15. 字符串的比较

```
function eatBLT(string sandwich) public {
  // 看清楚了，当我们比较字符串的时候，需要比较他们的 keccak256 哈希码
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```

## 16. Ownable Contracts 声明合约所属权（开发模板）

OpenZeppelin库的Ownable 合约

下面是一个 Ownable 合约的例子： 来自 _ OpenZeppelin _ Solidity 库的 Ownable 合约。 OpenZeppelin 是主打安保和社区审查的智能合约库，您可以在自己的 DApps中引用。等把这一课学完，您不要催我们发布下一课，最好利用这个时间把 OpenZeppelin 的网站看看，保管您会学到很多东西！

把楼下这个合约读读通，是不是还有些没见过代码？别担心，我们随后会解释。

```
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

下面有没有您没学过的东东？

构造函数：function Ownable()是一个 _ constructor_ (构造函数)，构造函数不是必须的，它与合约同名，构造函数一生中唯一的一次执行，就是在合约最初被创建的时候。

函数修饰符：modifier onlyOwner()。 修饰符跟函数很类似，不过是用来修饰其他已有函数用的， 在其他语句执行前，为它检查下先验条件。 在这个例子中，我们就可以写个修饰符 onlyOwner 检查下调用者，确保只有合约的主人才能运行本函数。我们下一章中会详细讲述修饰符，以及那个奇怪的_;。

indexed 关键字：别担心，我们还用不到它。

所以Ownable 合约基本都会这么干：

1. 合约创建，构造函数先行，将其 owner 设置为msg.sender（其部署者）

2. 为它加上一个修饰符 onlyOwner，它会限制陌生人的访问，将访问某些函数的权限锁定在 owner 上。

3. 允许将合约所有权转让给他人。

onlyOwner 简直人见人爱，大多数人开发自己的 Solidity DApps，都是从复制/粘贴 Ownable 开始的，从它再继承出的子类，并在之上进行功能开发。

既然我们想把 setKittyContractAddress 限制为 onlyOwner ，我们也要做同样的事情。

## 17. onlyOwner 函数修饰符（合约主人才能调用函数）

函数修饰符看起来跟函数没什么不同，不过关键字modifier 告诉编译器，这是个modifier(修饰符)，而不是个function(函数)。它不能像函数那样被直接调用，只能被添加到函数定义的末尾，用以改变函数的行为。

咱们仔细读读 onlyOwner:

```
/**
 * @dev 调用者不是‘主人’，就会抛出异常
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
```

onlyOwner 函数修饰符是这么用的：

```
contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  //注意！ `onlyOwner`上场 :
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```

注意 likeABoss 函数上的 onlyOwner 修饰符。 当你调用 likeABoss 时，首先执行 onlyOwner 中的代码， 执行到 onlyOwner 中的 _; 语句时，程序再返回并执行 likeABoss 中的代码。

可见，尽管函数修饰符也可以应用到各种场合，但最常见的还是放在函数执行之前添加快速的 require检查。

因为给函数添加了修饰符 onlyOwner，使得唯有合约的主人（也就是部署者）才能调用它。

```
注意：主人对合约享有的特权当然是正当的，不过也可能被恶意使用。比如，万一，主人添加了个后门，允许他偷走别人的僵尸呢？

所以非常重要的是，部署在以太坊上的 DApp，并不能保证它真正做到去中心，你需要阅读并理解它的源代码，才能防止其中没有被部署者恶意植入后门；作为开发人员，如何做到既要给自己留下修复 bug 的余地，又要尽量地放权给使用者，以便让他们放心你，从而愿意把数据放在你的 DApp 中，这确实需要个微妙的平衡。
```

## 18. 省gas的招数：结构封装（Struct packing）

在第1课中，我们提到除了基本版的 uint 外，还有其他变种 uint：uint8，uint16，uint32等。

通常情况下我们不会考虑使用 uint 变种，因为无论如何定义 uint的大小，Solidity 为它保留256位的存储空间。例如，使用 uint8 而不是uint（uint256）不会为你节省任何 gas。

除非，把 uint 绑定到 struct 里面。

如果一个 struct 中有多个 uint，则尽可能使用较小的 uint, Solidity 会将这些 uint 打包在一起，从而占用较少的存储空间。例如：

```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// 因为使用了结构打包，`mini` 比 `normal` 占用的空间更少
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

所以，当 uint 定义在一个 struct 中的时候，尽量使用最小的整数子类型以节约空间。 并且把同样类型的变量放一起（即在 struct 中将把变量按照类型依次放置），这样 Solidity 可以将存储空间最小化。例如，有两个 struct：

uint c; uint32 a; uint32 b; 和 uint32 a; uint c; uint32 b;

前者比后者需要的gas更少，因为前者把uint32放一起了。

## 19. 时间单位

变量 now 将返回当前的unix时间戳（自1970年1月1日以来经过的秒数）

Solidity 还包含秒(seconds)，分钟(minutes)，小时(hours)，天(days)，周(weeks) 和 年(years) 等时间单位。它们都会转换成对应的秒数放入 uint 中。所以 1分钟 就是 60，1小时是 3600（60秒×60分钟），1天是86400（24小时×60分钟×60秒），以此类推。

下面是一些使用时间单位的实用案例：

```
uint lastUpdated;

// 将‘上次更新时间’ 设置为 ‘现在’
function updateTimestamp() public {
  lastUpdated = now;
}

// 如果到上次`updateTimestamp` 超过5分钟，返回 'true'
// 不到5分钟返回 'false'
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

## 20. 公有函数和安全性

你必须仔细地检查所有声明为 public 和 external的函数，一个个排除用户滥用它们的可能，谨防安全漏洞。请记住，如果这些函数没有类似 onlyOwner 这样的函数修饰符，用户能利用各种可能的参数去调用它们。

## 21. 带参数的函数修饰符

之前我们已经读过一个简单的函数修饰符了：onlyOwner。函数修饰符也可以带参数。例如：

```
// 存储用户年龄的映射
mapping (uint => uint) public age;

// 限定用户年龄的修饰符
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// 必须年满16周岁才允许开车 (至少在美国是这样的).
// 我们可以用如下参数调用`olderThan` 修饰符:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // 其余的程序逻辑
}
```

看到了吧， olderThan 修饰符可以像函数一样接收参数，是“宿主”函数 driveCar 把参数传递给它的修饰符的。

来，我们自己生产一个修饰符，通过传入的level参数来限制僵尸使用某些特殊功能。

## 22. storage属性是在程序结束时进行存储的

## 23. 可支付 修饰符payable

这就允许出现很多有趣的逻辑， 比如向一个合约要求支付一定的钱来运行一个函数。

```
contract OnlineStore {
  function buySomething() external payable {
    // 检查以确定0.001以太发送出去来运行函数:
    require(msg.value == 0.001 ether);
    // 如果为真，一些用来向函数调用者发送数字内容的逻辑
    transferThing(msg.sender);
  }
}
```

在这里，msg.value 是一种可以查看向合约发送了多少以太的方法，另外 ether 是一个內建单元。

这里发生的事是，一些人会从 web3.js 调用这个函数 (从DApp的前端)， 像这样 :

```
// 假设 `OnlineStore` 在以太坊上指向你的合约:
OnlineStore.buySomething().send(from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001))
```

## 24. 提现 transfer

你可以通过 transfer 向任何以太坊地址付钱。 比如，你可以有一个函数在 msg.sender 超额付款的时候给他们退钱：

```
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```

## 25. 随机数函数

Solidity 中最好的随机数生成器是 keccak256 哈希函数.

我们可以这样来生成一些随机数

```
// 生成一个0到100的随机数:
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```

这个方法首先拿到 now 的时间戳、 msg.sender、 以及一个自增数 nonce （一个仅会被使用一次的数，这样我们就不会对相同的输入值调用一次以上哈希函数了）。

然后利用 keccak 把输入的值转变为一个哈希值, 再将哈希值转换为 uint, 然后利用 % 100 来取最后两位, 就生成了一个0到100之间随机数了。

## 26. ERC20 代币

一个 _代币_ 在以太坊基本上就是一个遵循一些共同规则的智能合约——即它实现了所有其他代币合约共享的一组标准函数，例如 transfer(address _to, uint256 _value) 和 balanceOf(address _owner).

在智能合约内部，通常有一个映射， mapping(address => uint256) balances，用于追踪每个地址还有多少余额。

所以基本上一个代币只是一个追踪谁拥有多少该代币的合约，和一些可以让那些用户将他们的代币转移到其他地址的函数。

## 27. ERC721 代币

ERC721同样是一个代币标准，此代币英文是Non-Fungible Tokens，简写为NFT，即非同质代币

僵尸不像货币可以分割 —— 我可以发给你 0.237 以太，但是转移给你 0.237 的僵尸听起来就有些搞笑。

其次，并不是所有僵尸都是平等的。 你的2级僵尸"Steve"完全不能等同于我732级的僵尸"H4XF13LD MORRIS 💯💯😎💯💯"。（你差得远呢，Steve）。

有另一个代币标准更适合如 CryptoZombies 这样的加密收藏品——它们被称为ERC721 代币.

ERC721 代币是不能互换的，因为每个代币都被认为是唯一且不可分割的。 你只能以整个单位交易它们，并且每个单位都有唯一的 ID。 这些特性正好让我们的僵尸可以用来交易。

## 28. ERC721 标准

让我们来看一看 ERC721 标准：

```
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```

开发过程中我们可以使用多继承，并且Solidity已经提供了 `erc721.sol` 的库：

```
pragma solidity ^0.4.19;

import "./zombieattack.sol";
import "./erc721.sol";

// 在这里声明 ERC721 的继承
contract ZombieOwnership is ZombieAttack,ERC721 {

}
```

## 29. balanceOf和ownerOf

balanceOf和ownerOf 是ERC721合约要求遵守的方法，只要合约遵守 ERC721，那么我们就可以和其它的ERC721合约进行交互，就像都是人类，人类之间就可以沟通，但人没法和狗沟通一样。

### （1）balanceOf
```
  function balanceOf(address _owner) public view returns (uint256 _balance);
```

这个函数只需要一个传入 address 参数，然后返回这个 address 拥有多少代币。

### （2）ownerOf

```
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
```

这个函数需要传入一个代币 ID 作为参数 (我们的情况就是一个僵尸 ID)，然后返回该代币拥有者的 address。

## 30. ERC721：转移标准

注意 ERC721 规范有两种不同的方法来转移代币：

```
function transfer(address _to, uint256 _tokenId) public;

function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```

第一种方法是代币的拥有者调用transfer 方法，传入他想转移到的 address 和他想转移的代币的 _tokenId。

第二种方法是代币拥有者首先调用 approve，然后传入与以上相同的参数。接着，该合约会存储谁被允许提取代币，通常存储到一个 mapping (uint256 => address) 里。然后，当有人调用 takeOwnership 时，合约会检查 msg.sender 是否得到拥有者的批准来提取代币，如果是，则将代币转移给他。

你注意到了吗，transfer 和 takeOwnership 都将包含相同的转移逻辑，只是以相反的顺序。 （一种情况是代币的发送者调用函数；另一种情况是代币的接收者调用它）。

所以我们把这个逻辑抽象成它自己的私有函数 _transfer，然后由这两个函数来调用它。 这样我们就不用写重复的代码了。

## 31. 合约安全增强：溢出和下溢

### （1）举例

假设我们有一个 uint8, 只能存储8 bit数据。这意味着我们能存储的最大数字就是二进制 11111111 (或者说十进制的 2^8 - 1 = 255).

来看看下面的代码。最后 number 将会是什么值？

```
uint8 number = 255;
number++;
```

在这个例子中，我们导致了溢出 — 虽然我们加了1， 但是 number 出乎意料地等于 0了。 (如果你给二进制 11111111 加1, 它将被重置为 00000000，就像钟表从 23:59 走向 00:00)。

下溢(underflow)也类似，如果你从一个等于 0 的 uint8 减去 1, 它将变成 255 (因为 uint 是无符号的，其不能等于负数)。

虽然我们在这里不使用 uint8，而且每次给一个 uint256 加 1 也不太可能溢出 (2^256 真的是一个很大的数了)，在我们的合约中添加一些保护机制依然是非常有必要的，以防我们的 DApp 以后出现什么异常情况。

### （2）使用 SafeMath

为了防止这些情况，OpenZeppelin 建立了一个叫做 SafeMath 的 _库_(library)，默认情况下可以防止这些问题。

不过在我们使用之前…… 什么叫做库?

一个_库_ 是 Solidity 中一种特殊的合约。其中一个有用的功能是给原始数据类型增加一些方法。

比如，使用 SafeMath 库的时候，我们将使用 using SafeMath for uint256 这样的语法。 SafeMath 库有四个方法 — add， sub， mul， 以及 div。现在我们可以这样来让 uint256 调用这些方法：

```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```

### （3）SafeMath 实现

来看看 SafeMath 的部分代码:

```
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```

首先我们有了 library 关键字 — 库和 合约很相似，但是又有一些不同。 就我们的目的而言，库允许我们使用 using 关键字，它可以自动把库的所有方法添加给一个数据类型：

```
using SafeMath for uint;
// 这下我们可以为任何 uint 调用这些方法了
uint test = 2;
test = test.mul(3); // test 等于 6 了
test = test.add(5); // test 等于 11 了
```

注意 mul 和 add 其实都需要两个参数。 在我们声明了 using SafeMath for uint 后，我们用来调用这些方法的 uint 就自动被作为第一个参数传递进去了(在此例中就是 test)

我们来看看 add 的源代码看 SafeMath 做了什么:

```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}
```

基本上 add 只是像 + 一样对两个 uint 相加， 但是它用一个 assert 语句来确保结果大于 a。这样就防止了溢出。

assert 和 require 相似，若结果为否它就会抛出错误。 assert 和 require 区别在于，require 若失败则会返还给用户剩下的 gas， assert 则不会。所以大部分情况下，你写代码的时候会比较喜欢 require，assert 只在代码可能出现严重错误的时候使用，比如 uint 溢出。

所以简而言之， SafeMath 的 add， sub， mul， 和 div 方法只做简单的四则运算，然后在发生溢出或下溢的时候抛出错误。

### （4）uint 256、16、32 的 SafeMath

我们遇到个小问题 — winCount 和 lossCount 是 uint16， 而 level 是 uint32。 所以如果我们用这些作为参数传入 SafeMath 的 add 方法。 它实际上并不会防止溢出，因为它会把这些变量都转换成 uint256:

```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}

// 如果我们在`uint8` 上调用 `.add`。它将会被转换成 `uint256`.
// 所以它不会在 2^8 时溢出，因为 256 是一个有效的 `uint256`.
```

这就意味着，我们需要再实现两个库来防止 uint16 和 uint32 溢出或下溢。我们可以将其命名为 SafeMath16 和 SafeMath32。

代码将和 SafeMath 完全相同，除了所有的 uint256 实例都将被替换成 uint32 或 uint16。

```
  using SafeMath for uint256;
  // 1. 为 uint32 声明 使用 SafeMath32
  using SafeMath32 for uint32;
  // 2. 为 uint16 声明 使用 SafeMath16
  using SafeMath16 for uint16;
```

## 32. natspec注释格式

Solidity 社区所使用的一个标准是使用一种被称作 natspec 的格式，看起来像这样：

```
/// @title 一个简单的基础运算合约
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice 现在，这个合约只添加一个乘法
contract Math {
  /// @notice 两个数相乘
  /// @param x 第一个 uint
  /// @param y  第二个 uint
  /// @return z  (x * y) 的结果
  /// @dev 现在这个方法不检查溢出
  function multiply(uint x, uint y) returns (uint z) {
    // 这只是个普通的注释，不会被 natspec 解释
    z = x * y;
  }
}
```

@title（标题） 和 @author （作者）很直接了.

@notice （须知）向 用户 解释这个方法或者合约是做什么的。 @dev （开发者） 是向开发者解释更多的细节。

@param （参数）和 @return （返回） 用来描述这个方法需要传入什么参数以及返回什么值。

注意你并不需要每次都用上所有的标签，它们都是可选的。不过最少，写下一个 @dev 注释来解释每个方法是做什么的。

——————————————————区块链和前端的交互 Web3.js——————————————————

## 33. Web3.js 是以太坊基金发布的 JavaScript 库

以太坊节点只能识别一种叫做 JSON-RPC 的语言。这种语言直接读起来并不好懂。当你你想调用一个合约的方法的时候，需要发送的查询语句将会是这样的：

```
// 哈……祝你写所有这样的函数调用的时候都一次通过
// 往右边拉…… ==>
{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}
```

幸运的是 Web3.js 把这些令人讨厌的查询语句都隐藏起来了， 所以你只需要与方便易懂的 JavaScript 界面进行交互即可。

你不需要构建上面的查询语句，在你的代码中调用一个函数看起来将是这样：

```
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔")
  .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```

## 34. Web3 Provider 和 Infura

来初始化它然后和区块链对话吧。

首先我们需要 Web3 Provider.

要记住，以太坊是由共享同一份数据的相同拷贝的 _节点_ 构成的。 在 Web3.js 里设置 Web3 的 Provider（提供者） 告诉我们的代码应该和 哪个节点 交互来处理我们的读写。这就好像在传统的 Web 应用程序中为你的 API 调用设置远程 Web 服务器的网址。

你可以运行你自己的以太坊节点来作为 Provider。 不过，有一个第三方的服务，可以让你的生活变得轻松点，让你不必为了给你的用户提供DApp而维护一个以太坊节点— Infura.

### （1）Infura

Infura 是一个服务，它维护了很多以太坊节点并提供了一个缓存层来实现高速读取。你可以用他们的 API 来免费访问这个服务。 用 Infura 作为节点提供者，你可以不用自己运营节点就能很可靠地向以太坊发送、接收信息。

你可以通过这样把 Infura 作为你的 Web3 节点提供者：

```
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```

不过，因为我们的 DApp 将被很多人使用，这些用户不单会从区块链读取信息，还会向区块链 _写_ 入信息，我们需要用一个方法让用户可以用他们的私钥给事务签名。

```
注意: 以太坊 (以及通常意义上的 blockchains )使用一个公钥/私钥对来对给事务做数字签名。把它想成一个数字签名的异常安全的密码。这样当我修改区块链上的数据的时候，我可以用我的公钥来 证明 我就是签名的那个。但是因为没人知道我的私钥，所以没人能伪造我的事务。
```

加密学非常复杂，所以除非你是个专家并且的确知道自己在做什么，你最好不要在你应用的前端中管理你用户的私钥。

不过幸运的是，你并不需要，已经有可以帮你处理这件事的服务了： Metamask.

### （2）Metamask

Metamask 是 Chrome 和 Firefox 的浏览器扩展， 它能让用户安全地维护他们的以太坊账户和私钥， 并用他们的账户和使用 Web3.js 的网站互动（如果你还没用过它，你肯定会想去安装的——这样你的浏览器就能使用 Web3.js 了，然后你就可以和任何与以太坊区块链通信的网站交互了）

作为开发者，如果你想让用户从他们的浏览器里通过网站和你的DApp交互（就像我们在 CryptoZombies 游戏里一样），你肯定会想要兼容 Metamask 的。

```
注意: Metamask 默认使用 Infura 的服务器做为 web3 提供者。 就像我们上面做的那样。不过它还为用户提供了选择他们自己 Web3 提供者的选项。所以使用 Metamask 的 web3 提供者，你就给了用户选择权，而自己无需操心这一块。
```

### （3）使用 Metamask 的 web3 提供者

Metamask 把它的 web3 提供者注入到浏览器的全局 JavaScript对象web3中。所以你的应用可以检查 web3 是否存在。若存在就使用 web3.currentProvider 作为它的提供者。

这里是一些 Metamask 提供的示例代码，用来检查用户是否安装了MetaMask，如果没有安装就告诉用户需要安装MetaMask来使用我们的应用。

```
window.addEventListener('load', function() {

  // 检查web3是否已经注入到(Mist/MetaMask)
  if (typeof web3 !== 'undefined') {
    // 使用 Mist/MetaMask 的提供者
    web3js = new Web3(web3.currentProvider);
  } else {
    // 处理用户没安装的情况， 比如显示一个消息
    // 告诉他们要安装 MetaMask 来使用我们的应用
  }

  // 现在你可以启动你的应用并自由访问 Web3.js:
  startApp()

})
```

你可以在你所有的应用中使用这段样板代码，好检查用户是否安装以及告诉用户安装 MetaMask。

```
注意: 除了MetaMask，你的用户也可能在使用其他他的私钥管理应用，比如 Mist 浏览器。不过，它们都实现了相同的模式来注入 web3 变量。所以我这里描述的方法对两者是通用的。
```

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)