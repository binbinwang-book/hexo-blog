---
title: JavaScript 学习笔记
date: 2021-07-11 22:14
categories: 大前端
tags: [JavaScript]
---

# JavaScript 学习笔记

## 一、起源

JavaScript 诞生于 1995年，它的出现主要是用于处理网页中的前端验证。

所谓的前端验证，就是指检查用户输入的内容是否符合一定的规则。

比如：用户名的长度、密码的长度、邮箱的格式等。

原名是由 Netscape 先开发的，叫 liveScrip；后面由 Sun 公司进入改成了 JavaScript

现在网速已经非常快了，所有校验放服务器也是可以的，所以目前为止 JavaScript 已经不局限于此了。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gri9wyo97pj30ib0b4tbv.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gri9y12ynhj30hy08yq58.jpg)

不同的浏览器使用的引擎是不同的，也就是实现 ECMAScript （读音：艾克马 Script）的方式不一样。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gria1af336j30ja0b1ju3.jpg)


JavaScript 主要是前端的

- ECMAScript：标准

- DOM：文档对象模型，提供一组对象可以操作网页

- BOM：提供一组对象可以操作浏览器

## 二、JS 的特点

- 解释型语言：不用编译，直接可以运行
- 动态语言
- 基于原型的面向对象

## 三、基本语法

（1）JS中严格区分大小写

（2）JS中每一条语句以分号(;)结尾

（3）JS中会忽略多个空格和换行，所以可以利用空格和换行对代码进行格式化

## 四、字面量和变量

（一）字面量：不可改变的值，可以理解为常量，可以直接使用 "helloWorld"

（二）变量：变量可以用来保存字面量，而且变量的值是可以任意改变的，变量更加方便使用

// 在js中使用var关键字来声明一个变量

var a;

// 为变量赋值

a = 123;

// 声明和赋值同时进行 

var a = 789;

## 五、标识符

（一）标识符：在JS中所有的可以由我们自主命名的都可以称为是标识符

例如：变量名、函数名、属性名 都属于标识符

1. 标识符中可以含有字母、数字、_、$
2. 标识符不能以数字开头
3. 标识符不能是ES中的关键字或保留字

![](https://tva1.sinaimg.cn/large/008i3skNgy1griazypyiuj30ia0cngpd.jpg)

关键字比如（var、if）：var var = 123; 是不被允许的

4. 标识符一般都采用驼峰命名法


（二）JS与Unicode（utf-8）的关系

JS底层保持标识符时实际上采用的是Unicode编码（utf-8），所有理论上讲，所有的utf-8中含有的内容都可以作为标识符，比如中文，但不建议用中文。

var 你好 = 789;

consolo.log(你好);

## 六、字符串

（一）数据类型

数据类型指的就是 字面量 的类型，在JS中一共有六种数据类型：

- String 字符串
- Number 数值
- Boolean 布尔值
- Null 空值
- Undefined 未定义
- Object 对象

1. 基本数据类型 vs. 引用数据类型
（1）String、Number、Boolean、Null、Undefined 基本数据类型
（2）Object属于引用数据类型

2. 字符串：在JS中字符串需要使用引号（双引号/单引号都可以）引起来

var str = "hello";

str = '我说："今天天气真不错!"'

str = "我说：\"今天天气真不错!\""

## 七、Number

在JS中所有的数值都是 Number 类型，包括 正数和浮点数

（一）使用typeof检查变量类型

语法：typeof 变量

var a = 123;

console.log(typeof a)

（二）JS中可以表示数字的最大值：Number.MAX_VALUE

如果使用 Number 表示的数字超过了最大值，则会返回一个 Infinity ，表示正无穷

Infinity 就是个字面量（不是字符串，而是 Number类型），专门用来保存正无穷的。

- Infinity 表示负无穷

（三）NaN 是一个特殊的数字，表示 Not A Number ，出现这个数值就是代码有bug了。

比如： console.log("abc" * "bcd")

输出结果就是 NaN

（四）Number.MIN_VALUE 表示0以上的最小值，正个正数，5e-324

（五）在JS中整数的运算基本可以保持精确

（六）如果使用JS进行浮点数运算，可能得到一个不精确的结果

我们的所有运算都是要转成二进制的，但浮点数是没办法直接用二进制精确表示的，所以处理的结果可能会有小数点，所以千万不要使用JS进行对精确度要求比较高的运算（可以放到服务器计算）：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gric44mgnhj30ao070406.jpg)

## 八、布尔值

（一）true / false

var bool = true;

## 九、Null 和 Undefined

（一）Null 

1. null类型的值只有一个，就是 null

var a = null;

2. null 专门用来表示一个为空的对象

使用typeof检查null时，会返回 object 类型

（二）undefined 类型的值只有一个，就是 undefined

当生命一个变量，但并不给并变量赋值时，它的值就是 undefined。

使用 typeof 检查一个 undefined 时也会返回 undefined。

var a;

console.log(typeof a)

## 十、强制类型转换

（一）将一个数据类型强制转换为其它数据类型

（二）类型转换主要指，将其它的数据类型，转换为：String、Number、Boolean，一般不会转成 Null 和 undefined。

（三）将其它数据类型转换为 String

var a = 123;

console.log(typeof a);

1. 方式一：调用被转换数据类型的toString()方法，该方法不会影响到原变量

但是注意：null 和 undefined 这两个值没有toString()方法，如果调用它俩的toString()方法，会报错

var a = 123;

// 调用 a 的 toString() 方法

var b = a.toString();

a = a.toString();

console.log(typeof b)

2. 方式二： 调用String()函数，并将被转换的数据作为参数传递给函数

对于 null 和 undefined，就不会调用toString()方法，它会将 null 直接转换为 "null"

var a = 123;

a = String(a);

console.log(a);

## 十一、将其它数据类型转换为 Number

（一）字符串 -> 数字

1. 方式一：

（1）纯数字字符串，则直接将其转换为数字

var a = "123";

a = Number(a);

console.log(typeof a)

（2）如果字符串中有非数字的内容，则转换为NaN

var a = "abc";

a = Number(a);

console.log(typeof a)  ===>  NaN

（3）如果字符串是一个空串或者是一个全是空格的字符串，则转换为0

2. 方式二：

这种方式专门用来对付字符串

（1）parseInt() 把一个字符串转换成一个整数，可以将一个字符串中有效的整数内容取出来，然后转换为Number

（2）parseFloat() 把一个字符串转换为一个浮点数

（3）如果对非String使用parseInt()或parseFloat()，它会先将其转换为String，然后再操作

```
var a = "123px";
// 调用parseInt()函数将a转换为Number
a = parseInt(a);
console.log(a) ===> 123

a = "123a778b"
a = parseInt(a);
console.log(a) ===> 123

a = "d123ks"
a = parseInt(a);
console.log(a) ===> NaN

a = "123.23b"
a = parseInt(a);
console.log(a) ===> 123

a = "123.23b"
a = parseFloat(a);
console.log(a) ===> 123.23

a = true;
a = parseInt(a);
console.log(a) ===> NaN
```

（二）布尔值 -> 数字

1. true 转成 1
2. false 转成 0 

（三）null -> 0

（四）undefined -> NaN

## 十二、转换为Boolean

（一）数字 -> 布尔
除了0和NaN，其余都是true

var a = 123;

a = Boolean(a);  ==> true

a = -123;

a = Boolean(a);  ==> true

a = 0;

a = Boolean(a);  ==> false

（二）字符串 -> 布尔
除了空串，其余的都是true

（三）null和undefined都会转换为false

## 十三、算数运算符

运算符也叫操作符

通过运算符可以对一个或多个值进行运算

（一）typeof

比如：typeof就是运算符，可以来获得一个值的类型，它会将该值的类型以字符串的形式返回

```
var a = 123;

var res = typeof a;

console.log(res)
```

（二）算数运算符

对非Number类型的值进行运算时，会先将这些值转成Number，然后再进行运算（除了加法和字符串相加的情况，其余的全部都转成 Number）：可以通过这个特点做隐式的类型转换，可以通过为一个值 -0 * 1 / 1 将其转换为Number。

1. 加法 + 

（1）任何值和NaN做运算，都得NaN

（2）字符串拼串

a.如果对两个字符串进行相加，那么是字符串拼串的操作

b.任何值和字符串做加法运算，都会先把值转换为字符串，然后拼串

c. 可以利用这一特点，可以将任意的数据类型转换为String，只需要为任意类型的数据加一个空串 "" 即可转换为String

res = 1 + 2 + "3";  ==> "33"

res = "1" + 2 + 3;  ==> "123" 由左向右进行计算

2. 减法(-)、乘法(*)、除法(/)

res = 100 - "2";  ==> 98 Number

res = 2 * "8";    ==> 16 Number

res = 3 / 2       ==> 1.5

3. 取模 %

## 十四、一元运算符

一元运算符，值需要一个操作数， 1+1 是两元运算符，typeof 是一元运算符。

+ 正号

- 负号

可以对一个其他的数据类型使用 + ，来将其转换为 Number （隐式类型转换）。

## 十五、自增和自减

（一）自增 ++

自增分成两种：后++（a++）和前++(++a)

无论是 a++ 还是 ++a ，都会立即使自变量增加1

```
a = 1
console.log(a++); ==> 1 先使用，再运算
console.log(a);   ==> 2

a = 1
console.log(++a); ==> 2 先运算，再使用
console.log(a);   ==> 2

var c = 10;
c++
console.log(c++) ==> 11，先输出c，再++
```

（二）自减 --

## 十六、逻辑运算符

（一）！非

（二）&&与

如果第一个值为false，则不会继续判断

（三）|| 或

第一个值为 false， 则会检查第二个值

第一个值为 true， 则不会检查第二个值

（四） &&  ||

1. 对于非布尔值进行 与或 运算时

会先将其转换为布尔值，然后再运算，并且返回原值

2. 与运算

（1）如果两个值都为 true，则返回后边的

（2）如果第一个值为false，则直接返回第一个值

var result = NaN && 0 ==> NaN

3. 或运算

（1）如果第一个值为 true ，则直接返回第一个值

（2）如果第一个值为false，则直接返回第二个值

var res = "" || "hello";  ==> hello

res = -1 || "你好"   ==> -1

## 十七、赋值运算符

1.任何值和 NaN 做任何比较都是 false

2.如果符号两侧的值都是字符串，不会将其转换为数字进行比较，而会分别比较字符串中 Unicode 中编码，比较字符编码时，是一位一位进行比较,如果两位一样，则比较下一位，所以借用它来对英文进行排序。

比较两个字符型的数字时，一定要转型

console.log("11" < "5")  ==> true

console.log(+"11" < +"5")  ==> true

比较中文时没有意义。

## 十八、相等运算符

（一）== 

当使用 == 来比较两个值时，如果值的类型不同，则会自动进行类型转换，将其转换为相同的类型，然后再比较。

console.log("1" == 1)  ==> true

console.log(true == "1")  ==> true

console.log(true == "hello") ==> false

console.log(null == 0)  ==> false 

```
/*
	undefined 衍生自 null
	所以这两个值做相等判断时，会返回true
	console.log(undefined == null) ;
	
	NaN 不合任何值相等，包括它本身
	console.log(NaN == NaN) ==> false
*/
```

需求：判断b是否是 NaN

可以通过isNaN()函数来判断一个值是否是NaN

console.log(isNan(b))

（二）判断不相等 !=

（三）=== 全等

用来判断两个值是否全等，它和相等类型，不同的是它不会做自动的类型转换

如果两个值的类型不同，直接返回false

console.log("123" === 123)  ==> false

console.log(null == undefined) ==> true

console.log(null === undefined) ==> false

（四）!===

## 十九、条件运算符（三元运算符）

```
var a = 30;

var b = 20;

a > b ? alert("a大"):alert("b大")

var max = a > b ? a : b;

"hello" ? alert("a大"):alert("b大")
```

## 二十、运算符的优先级

（一）逗号，

使用，可以分割多个语句，一般可以声明多个变量时使用

var a = 1, b = 2, c = 3;

就和数学中一样，在JS中运算符也有优先级，

## 二十一、代码块

```
/*
	我们的程序是由一条一条语句构成的，
	语句是按照自上而下的顺序一条一条之星的
	在JS可以使用 {}来为语句进行分组：
		它们要哦都执行，要么都不执行，
		一个{}中的语句我们称为一个代码块
		
	JS中的代码块，只具有分组的作用，没有其它的用途
*/

{
	alert("hello");
	console.log("你好")
}

```

## 二十二、if语句

```
if (a > 10) {
	alert(@"hello world");
}

if (a > 10 && a < 20) {
	alert(@"hello world");
}


if (a > 10 && a < 20) {
	alert(@"hello world");
} else if(a > 20 && a < 30){
	alert(@"hello ~~");
} else {
	alert(@"world ~~");
}
```

## 二十三、对象的简介

（一）对象的分类

1. 内建对象

（1）由ES标准定义的对象，在任何的ES的实现中都可以使用

（2）比如：Math String Number Boolean Function Object

2. 宿主对象

（1）由JS的运行环境提供的对象，目前来讲主要是指由浏览器提供的对象

（2）比如 BOM DOM

3. 自定义对象

（1）由开发人员自己创建的对象

（二）对象的使用

1. 创建对象

// 使用new关键字调用的函数，是构造函数constructor

// 构造函数是专门用来创建对象的函数

var obj = new Object();

2. 在对象中保存的值为属性

obj.name = "孙悟空"

obj.gender = "男"

对象的属性名不强制要求标识符的规范去做，但我们还是尽量按照标识符的规范去做

如果要使用特殊的属性名，不能采用 . 的方式来操作，需要使用另外一种方式：

obj["123"] = 789;

console.log(obj["123"]);

3. 读取对象中的属性

console.log(obj.age)

// 如果读取对象中没有的属性，不会报错而是会返回 undefined

console.log(obj.hello)

使用[]取属性的值更加灵活：

var n = "123"

console.log(obj[n])

console.log(obj.test2) ==> undefined

4. 删除对象的属性

delete obj.name;

（三）属性名和属性值

1. 检查obj中是否有test2属性

console.log("test2" in obj)

2. 强引用问题

```
var obj = new Object();
obj.name = "孙悟空";

var obj2 = obj;

// 修改obj的name名

obj.name = "猪八戒";

console.log(obj.name)
console.log(obj2.name)
```

打印的结果都是 猪八戒 

## 二十三、对象字面量

（一）创建一个对象

var obj = new Object();

var obj = {}; // 也是创建了一个对象

（二）使用对象字面量，可以在创建对象时，直接指定对象中的属性

语法：{属性名：属性值，属性名：属性值,...}

var obj2 = {
	name:"猪八戒",
	age:28,
	gender:"男"}
	
对象字面量的属性名可以加引号，也可以不加，建议不加（如果要使用一些特殊的名字，则必须用引号）

如果一个属性之后没有其它的属性了，就不用写逗号 ， 了

## 二十四、函数的简介

(一)封装函数

函数中可以封装一些功能（代码）

使用typeof检查一个函数对象时，会返回 function

var fun = new Function();

可以将要封装的带啊以字符串的形式传递给构造函数

// 在实际开发中很少使用构造函数来创建一个函数（比如下面的例子）

var fun = new Function("console.log("hello world");");

（二）调用函数

fun();

（三）使用函数声明创建函数

语法：
	function 函数名([形参1，形参2，...，形参N]) {
		语句...
	}
	
```
function fun2() {
	console.log(@"hello world");
}

console.log(fun2);
```

（四）使用函数表达式来创建一个函数

var 函数名 = function([形参1，形参2，...，形参N]) {
	语句...
}

## 二十五、函数的参数

```
function sum(a,b) {
	console.log(a+b);
}
```

在调用函数时，可以在()中指定实参（实际参数）

实参将会复制给函数中对应的形参：

sum(1,2);

## 二十六、函数的返回值 return 

创建一个函数，计算三个数的和

function sum(a ,c ,c) {
	var sum = a + b + c;
	return d;
}

var res = sum(4,7,8);

如果不加return，相当于返回一个 undefined

## 二十六、返回值的类型（可以是任意类型）

```
function fun2() {
	var obj = {name:"沙和尚"}
	return obj;
}
```

## 二十七、函数补充

对象的属性值可以是任何的数据类型，也可以是个函数

```
obj.sayName = function() {
	console.log(obj.name)
};

obj.sayName();
```

## 二十八、全局作用域

作用域：作用域指一个变量的作用的范围

在JS中一共有两种作用域：

（一）全局作用域

（二）函数作用域

## 二十九、this

```

function fun() {
	console.log(this);
}

直接调用 fun();  ===> 输出 [object Window]

``` 

解析器在调用函数每次都欧会向函数内部传递进一个隐含的参数

这个隐含的参数就是this，this指向的是一个对象，

这个对象我们称为函数执行的上下文对象，

根据函数的调用方式的不同，this会指向不同的对象

谁调用函数，this就指向谁，this是调用方法的引用。

```

function fun() {
	console.log(this);
}

// 创建一个对象
var obj = {
	name : "孙悟空",
	sayName : fun
};

obj.sayName();  ==> [object Object]

``` 

```

// 创建一个name变量
var name = "全局";

// 创建一个fun()函数
function fun() {
	console.log(name);
}

// 创建两个对象
var obj = {
	name = "孙悟空",
	sayName:fun;
};

var obj2 = {
	name = "沙和尚",
	sayName:fun;
};

fun(); ==> "全局"

// 我们希望调用obj.sayName()时可以输出obj的名字

function fun() {
	console.log(this.name);
}

```

## 三十、根据工厂方法创建对象

（一）通过方法创建对象

```
// 创建三个对象
var obj = {
	name = "孙悟空",
	sayName:fun;
};

var obj2 = {
	name = "沙和尚",
	sayName:fun;
};

var obj3 = {
	name = "猪八戒",
	sayName:fun;
};

```

上面这种方法创建对象过于麻烦，出现了大量重复性代码

```
// 通过该方法可以大批量创建对象
function createPerson(name,fun) {
	// 创建一个新的对象
	return {name:name,sayName:fun};
};

var obj4 = createPerson("小狗",fun)
```

（二）构造函数

创建一个构造函数，专门用来创建 Person 对象的，

构造函数就是一个普通的函数，创建方式和普通函数没有区别，

不同的是构造函数习惯上首字母大写。

构造函数和普通函数的区别就是调用方式的不同

普通函数是直接调用，而构造函数需要使用 new 关键词调用

```

function Person(name, age ,gender) {
	this.name = "孙悟空";
	this.age = 18;
	this.gender = "男";
	this.sayName = function() {
		alert(this.name);
	};
}

var per = new Person("孙悟空",18,"男");

```

构造函数的执行流程：

1. 立刻创建一个新的对象

2. 将新建的对象设置为函数中this，在构造函数中可以使用this来引用新建的对象

3. 逐行执行函数中的代码

4. 将新建的对象作为返回值返回

使用同一个构造函数创建的对象，我们称为一类对象

（三）instanceof 检查一个对象是否是一个类的实例

console.log (per instanceof Person) ==> true

console.log (per instanceof Object) ==> true

（四）this总结

1. 当以函数的形式调用时，this是Window

2. 当以方法的形式调用时，谁调用方法 this 就是谁

3. 当以构造函数的形式调用时，this 就是新创建的那个对象

（五）构造函数修改

```

function Person(name, age ,gender) {
	this.name = "孙悟空";
	this.age = 18;
	this.gender = "男";
	this.sayName = function() {
		alert(this.name);
	};
}

var per = new Person("孙悟空",18,"男");

```

创建一个 Person 构造函数：

为每一个对象都添加了一个 sayName 方法，也就是所有实例的 sayName 都是唯一的。

这样就导致了构造函数执行一次就会创建一个新的方法，执行10000次就会创建10000个新的方法

这是完全没有必要的，完全可以使所有对象共享同一个方法：

```

function Person(name, age ,gender) {
	this.name = "孙悟空";
	this.age = 18;
	this.gender = "男";
	this.sayName = fun;
}

function fun() {
	console.log(this.name);
}

var per = new Person("孙悟空",18,"男");

```

但直接把fun定义在全局作用域很不安全，下面做优化。

## 三十一、原型对象

（一）原型对象使用

将函数定义在全局作用域，污染了全局作用域的命名空间（别人不能叫同个方法了）

而且定义在全局作用域中也很不安全

我们所创建的每一个函数，解析器都会向函数中添加一个属性 prototype，

这个属性对应着一个对象，这个对象就是我们所谓的原型对象。

如果函数作为普通函数调用 prototype 没有任何作用，

当函数通过构造函数调用时，它所创建的对象都会有一个隐含的属性，

指向该构造函数的原型对象，我们可以通过 __proto__ 来访问该属性。

原型对象就相当于一个公共的区域，所有的同一个类的实例都可以访问到原型对象。

我们可以将对象中共有的内容，统一设置到原型对象中。

```
function MyClass() {
}

// 向MyClass的原型中添加属性a
MyClass.prototype.a = 123;

var mc = new MyClass();

var mc2 = new MyClass();

console.log(mc.a) ==> 123

mc.a = "我是mc";

console.log(mc.a) ==> "我是mc"

```

当我们访问对象的一个属性或方法时，它会先在对象自身中寻找，如果有则直接使用

如果没有则会去原型对象中寻找，如果找到则直接使用。

以后我们创建构造函数时，可以将这些对象共有的属性和方法统一添加到构造函数的原型对象中。

这样不用分别为每一个对象添加，也不会影响到全局作用域，就可以似每个对象都具有这些属性和方法了。


（二）检查自己的方法hasOwnProperty

使用 in 检查对象中是否含有某个属性时，如果对象中没有但原型中有，也会返回true

那检查自己的属性使用什么方法呢？

用 hasOwnProperty()

mc.hasOwnProperty("name")

（三）原型对象中也有原型对象（原型链）

原型对象也是对象，所以它也有原型，我们使用一个对象的属性或方法时，会先在自身中寻找，

自身中如果有，则直接使用，

如果没有则去原型对象中寻找，如果原型对象中有，则使用

如果没有则去原型的原型中寻找，直到找到 Object 对象的原型，

Object对象的原型没有原型，如果在Object中依然没有找到，则返回 null

console.(mc.__proto.__proto__); ==> undefined

console.(mc.__proto.__proto__.__proto__); ==> null

![](https://tva1.sinaimg.cn/large/008i3skNgy1grp1q9iqipj30q80ern1m.jpg)

## 三十二、覆写toString()方法


```

function Person(name, age ,gender) {
	this.name = "孙悟空";
	this.age = 18;
	this.gender = "男";
}

var per = new Person("孙悟空",18,"男");

console.log(per.toString());

```

当我们直接在页面中打印一个对象时，事实上输出的是对象 toString() 方法的返回值。

```

function Person(name, age ,gender) {
	this.name = "孙悟空";
	this.age = 18;
	this.gender = "男";
}

Person.prototype.toString = function() {
	return "Person[name =" + this.name + ",age = " + this.age + ",gender = " + this.gender;
};

var per = new Person("孙悟空",18,"男");

console.log(per.toString());

```

## 三十三、垃圾回收

![](https://tva1.sinaimg.cn/large/008i3skNgy1grp1rqj5h1j30q70dz0u0.jpg)

当一个对象没有任何的变量或属性对它进行引用，此时我们将永远无法操作该对象，此时这种对象就是一个垃圾。

这种对象过多会占用大量的内存空间，导致程序运行变慢，

所以这种垃圾必须进行清理。

在JS中拥有自动的垃圾回收机制，会自动将这些对象从内存中销毁，我们不需要也不能进行手动触发垃圾回收的操作。

## 三十四、数组

数组也是一个对象

它和我们普通对象功能类似，也是用来存储一些值的

不同的是普通对象是使用字符串作为属性名的，而数组是使用数字来作为索引操作元素

（一）创建 Array 对象的语法

new Array();

new Array(size);

new Array(element0, element1, ..., elementn);

（二）Array 对象方法

![](https://tva1.sinaimg.cn/large/008i3skNgy1grp210o3hij30ip0gy0v2.jpg)

（三）修改length

如果修改的 length 小于原长度，则多出的元素会被删除

（四）添加元素

向数组的最后一个位置添加元素

arr[arr.length] = "hello"

（五）创建一个数组

var arr = new Array();

var arr = [];

var arr = [1,2,3];

var arr = new Array(1,2,3);

var arr = [10]; // 创建一个数组中只有一个元素10

var arr = new Array(10); // 创建一个长度为10的数组


## 三十五、for循环

```
for (var i=0;i<cars.length;i++)
{ 
    document.write(cars[i] + "<br>");
}
```

## 三十六、while遍历

```
while (i<5)
{
    x=x + "The number is " + i + "<br>";
    i++;
}
```

## 三十七、数据转换方法

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsenlrdfpsj30lq0kfgo6.jpg)

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)