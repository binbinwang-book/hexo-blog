---
title: 我采用 UML 绘制技术流程图的原因
date: 2021-07-25 15:20
tags: [UML,沟通效率]
categories: [技术方案]
---

快速回忆的示例图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gthj36tuazj60o20jjta502.jpg)

# 我采用 UML 绘制技术流程图的原因

## 一、对提升技术方案沟通效率的思考

### 1. Show me the code.是不是一个好的提议？

作为程序员，我们经常听到的一句话是：Talk is cheap. Show me the code.

结合我日常工作的情况，这句话诞生于两个场景：

一是「方案讨论」中：B不认可A的方案，A既然说可行，那A直接写code吧，强调动手性。

二是「代码review」中：B不知道A在说什么，B打算直接看A的代码，强调沟通的效率性。

其实在「方案讨论」中，使用"Talk is cheap. Show me the code."其实不是一个好的方法，「先实现，再评估」是个浪费人力的事情，任何方案都应该先彻底评估完毕，然后再进行实施。

在「代码review」中，"Talk is cheap. Show me the code."确实是一个高效沟通的方式之一，但如果review的是一个比较复杂的需求变更，在没有有效沟通的场景下，review code实际上也只是管中窥豹，很难从架构上提出建议，"Talk is cheap. Show me the code."只是将「非code沟通」的工作后置到 「code review」过程中，埋下了很大的风险。

所以这里根本的问题是，为什么「技术方案沟通」是一个很多人头疼的问题，有什么办法解决这个问题？

### 2. 为什么「技术方案沟通」是一个问题

以A和B沟通为例，A阐述方案，B进行聆听。

这个过程中A以为自己表达清楚了，B仍旧稀里糊涂，为什么会这样呢？

经过我自己的亲身体会，这里可能有三个原因：

- 原因1. A预设B知道了某些前置条件，而实际上B并不知道
- 原因2. 对于同一件事情，A和B的表达/描述不同
- 原因3. 表述能力和口头理解能力，也会影响A和B之间的沟通

在 技术方案 讨论过程中，一个复杂的技术方案经常会有「原因1」和「原因2」的干扰，所以「描述前置条件」和「统一表达规范」是增强「技术方案沟通」效率的有效手段。

### 3. 为什么建议使用 UML

很早之前我进行技术方案的阐述时，已经发现了「描述前置条件」的重要性，所以我在绘制技术方案图时，都会阐明前置条件，如下图示例：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst6vkm6l3j30gb0jqgma.jpg)

然而我以为表达很清楚的技术方案图，但在别人浏览时也会产生一些流程上的疑问，问了几个朋友之后我发现在「统一表达规范」上出现了问题，即使流程图画得再漂亮，如果没有按照规范来进行制作，就相当于别人在理解我的流程图时，要理解我制作的流程图中「箭头、虚线、方框」的含义。

所以为了解决在技术流程图上「统一表达规范」的问题，我决定之后在技术方案描述中采用 UML，制作流程图，首先强调的是规范和专业，其次再是美观。


### 4.如何提升技术方案沟通效率？

大型改动，口头表述不清的技术方案，应该做好准备工作后再进行口头讲解，准备工作可以是：

- 技术方案issue
- 技术UML流程图
- 「技术方案issue」 && 「技术UML流程图」

不要直接把上千行的code change甩给 reviewer ，然后尝试用口头描述讲清楚变更的原因，这是对自己代码不负责，也是对 reviewe 流程不负责的表现。

## 二、UML 是什么

UML的全称Unified Modeling Language，即统一建模语言或标准建模语言，是始于1997年一个OMG标准，它是一个支持模型化和软件系统开发的图形化语言，为软件开发的所有阶段提供模型化和可视化支持，包括由需求分析到规格，到构造和配置。 

`面向对象的分析与设计(OOA&D，OOAD)方法`的发展在80年代末至90年代中出现了一个高潮，UML是这个高潮的产物。它不仅统一了Booch、Rumbaugh和Jacobson的表示方法，而且对其作了进一步的发展，并最终统一为大众所接受的标准建模语言。


### 1. UML图的特点

UML统一了各种方法对不同类型的系统、不同开发阶段以及不同内部概念的不同观点，从而有效的消除了各种建模语言之间不必要的差异。它实际上是一种通用的建模语言，可以为许多面向对象建模方法的用户广泛使用。
UML建模能力比其它面向对象建模方法更强。`它不仅适合于一般系统的开发，而且对并行、分布式系统的建模尤为适宜。`

`UML是一种建模语言，而不是一个开发过程。`

### 2. UML图的种类

截止UML2.0一共有13种图形（UML1.5定义了9种，2.0增加了4种）

分别是：

- 用例图、类图、对象图、状态图、活动图、顺序图、协作图、构件图、部署图9种
- 包图、时序图、组合结构图、交互概览图4种。

具体描述如下：

- 用例图：从用户角度描述系统功能。
- 类图：描述系统中类的静态结构。
- 对象图：系统中的多个对象在某一时刻的状态。
- 状态图：是描述状态到状态控制流，常用于动态特性建模
- 活动图：描述了业务实现用例的工作流程
- 顺序图：对象之间的动态合作关系，强调对象发送消息的顺序，同时显示对象之间的交互
- 协作图：描述对象之间的协助关系
- 构件图：一种特殊的UML图来描述系统的静态实现视图
- 部署图：定义系统中软硬件的物理体系结构
- 包图：对构成系统的模型元素进行分组整理的图
- 时序图： 表示生命线状态变化的图
- 组合结构图：表示类或者构建内部结构的图
- 交互概览图：用活动图来表示多个交互之间的控制关系的图

## 三、如何使用UML进行面向对象的设计

面向对象有三大基本特征：封装、继承和多态，这都可以使用 UML 进行描述。

### 1. 类 - Class

Java是一门面向对象语言，那最基础的就类了。类（Class）封装了数据和行为，是面向对象的重要组成部分，它是具有相同属性、操作、关系的对象集合的总称。在系统中，每个类都具有一定的职责，职责指的是类要完成什么样子的功能，要承担什么样子的义务。一个类可以有多种职责，但是设计得好的类一般只有一种职责。

假如我现在定义了这么一个类：

```
public class Person {
    private String name = "Jack";

    public String getName()     {
        return name;
    }

    public void setName(String name)     {
        this.name = name;
    }
    
    protected void playBasketball()     {
        pass();
    }
    
    private void pass()     {
    }
}
```

那么此类对应的UML为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst77wlptyj30bi07q74l.jpg)

看到该图分为三层：最顶层的为类名，中间层的为属性，最底层的为方法。

属性的表示方式为：【可见性】【属性名称】：【类型】={缺省值，可选}

方法的表示方式为：【可见性】【方法名称】（【参数列表】）：【类型】 

可见性都是一样的，"-"表示private、"+"表示public、"#"表示protected。

### 2. 继承 - Generalization

继承也叫作泛化（Generalization），用于描述父子类之间的关系，父类又称为基类或者超类，子类又称作派生类。在UML中，泛化关系用带空心三角形的实线来表示。

假如现在我又定义了一个Student和一个Teacher：

```
public class Student extends Person {
    private String studentNo;
    
    public void study()     {
    }
}
```

```
public class Teacher extends Person {
    private String teacherNo;
    
    public void teach()     {
    }
}
```

那么，用UML表示这种关系应当是：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst78xqurkj30ng0ec3zj.jpg)

### 3. 抽象继承 - abstract Generalization

上面的继承是普通的继承，在Java中，除了普通的继承之外，众所周知的还有一种抽象的继承关系，因此就再讲讲抽象继承关系，作为上面的继承的补充。

比方说我想实现一个链表（Link），插入（insert）与删除（remove）动作我想让子类去实现，链表本身只实现统计链表中元素个数的动作（count），然后有一个子类单向链表（OneWayLink）去实现父类没有实现的动作，Java代码为：

```
public abstract class Link
{
    public abstract void insert();
    public abstract void remove();
    
    public int count()
    {
        return 0;
    }
}
```

```
public class OneWayLink extends Link
{
    public void insert()
    {
        
    }

    public void remove()
    {
        
    }
}
```

其 UML 的画法为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7fqrexrj30720demxg.jpg)

在 UML 中，抽象类无论类名还是抽象方法名，都以`斜体`的方式表示，因为这也是一种继承关系，所以子类与父类通过带空心三角形的实线来联系。

### 4. 接口方法实现 - interface

很多面向对象的语言中都引入了接口的概念，如Java、C#等，在接口中通常没有属性，而且所有的操作都是抽象的，只有操作的声明没有操作的实现。UML中用与类类似的方法表示接口，假设我有一个Animal：

```
public interface Animal 
{
    public void move();
    public void eat();
}
```

那么它的UML应当表示为：

很简单，注意在方法上应当有<<interface>>表示这是一个接口。接口一般没有属性，所以这里中间层没有，有属性要注意也都是常量。

接下来，有一个Dog和一个Cat实现了Animal：

```
public class Dog implements Animal {
    public void move()     {
        
    }

    public void eat()     {
        
    }
}
```

```
public class Cat implements Animal {
    public void move()     {
        
    }

    public void eat()     {
        
    }
}
```

此时应当使用带空心三角形的虚线来表示，UML应该是这样的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7ipvkj1j30iq0c8aaj.jpg)

两个抽象方法，Dog和Cat的实现将不一样，当然，在Dog和Cat之中，也可以增加Dog和Cat自己的变量和方法。

### 5. 关联关系 - Assocition

关联（Assocition）关系是类与类之间最常见的一种关系，它是一种结构化的关系，表示一类对象与另一类对象之间有联系，如汽车和轮胎、师傅和徒弟、班级和学生等。在UML类图中，用实线连接有关联关系的对象所对应的类，在Java中通常将一个类的对象作为另一个类的成员变量。关联关系分单向关联、双向关联、自关联，逐一看一下。

#### （1）单向关联关系

单向关联指的是关联只有一个方向，比如顾客（Customer）拥有地址（Address），其Java实现为：

```
public class Address
{

}
```

```
public class Customer
{
    private Address address;
}
```

UML的画法为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7mkubilj30j20520st.jpg)

#### （2）双向关联关系

默认情况下的关联都是双向的，比如顾客（Customer）购买商品（Product），反之，卖出去的商品总是与某个顾客与之相关联，这就是双向关联。Java类的写法为：

```
public class Product
{
    private Customer customer;
}
```

```
public class Customer
{
    private Product[] product;
}
```

对应的UML类图应当是：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7nb78ufj30nu04et90.jpg)

#### （3）自关联关系

自关联，指的就是对象中的属性为对象本身，这在链表中非常常见，单向链表Node中会维护一个它的前驱Node，双向链表Node中会维护一个它的前驱Node和一个它的后继Node。就以单向链表为例，它的Java写法为：

```
public class Node
{
    private Node nextNode;
}
```

对应的UML类图应当是：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7pbt7hsj30cw06o3yj.jpg)

### 6. 聚合关系 - Aggregation

聚合（Aggregation）关系表示整体与部分的关系。在聚合关系中，成员对象是整体的一部分，但是成员对象可以脱离整体对象独立存在。在UML中，聚合关系用带空心菱形的直线表示，如汽车（Car）与引擎（Engine）、轮胎（Wheel）、车灯（Light），Java表示为：

```
public class Engine
{

}

public class Wheel
{

}

public class Light
{

}

public class Car
{
    private Engine engine;
    private Light light;
    private Wheel wheel;
    
    public Car(Engine engine, Light light, Wheel wheel)
    {
        super();
        this.engine = engine;
        this.light = light;
        this.wheel = wheel;
    }
    
    public void drive()
    {
        
    }
}
```

对应的UML类图为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7rmapjvj30gu0cut96.jpg)

代码实现聚合关系，成员对象通常以构造方法、Setter方法的方式注入到整体对象之中。

### 7. 组合关系 - Composition

组合（Composition）关系也表示的是一种整体和部分的关系，但是在组合关系中整体对象可以控制成员对象的生命周期，一旦整体对象不存在，成员对象也不存在，整体对象和成员对象之间具有同生共死的关系。在UML中组合关系用带实心菱形的直线表示。

比如人的头（Head）和嘴巴（Mouth）、鼻子（Nose），嘴巴和鼻子是头的组成部分之一，一旦头没了，嘴巴也没了，因此头和嘴巴、鼻子是组合关系，Java表示为：

```
public class Mouth
{

}

public class Nose
{

}

public class Head
{
    private Mouth mouth;
    private Nose nose;
    
    public Head()
    {
        mouth = new Mouth();
        nose = new Nose();
    }
    
    public void shake()
    {
        
    }
}
```

其UML的表示方法为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7sm0sloj30cu0aqglu.jpg)

代码实现组合关系，通常在整体类的构造方法中直接实例化成员类，这是因为组合关系的整体和部分是共生关系，如果通过外部注入，那么即使整体不存在，那么部分还是存在的，这就相当于变成了一种聚合关系了。

### 8. 依赖关系 - Dependency

依赖（Dependency）关系是一种使用关系，特定事物的改变有可能会影响到使用该事物的其他事物，在需要表示一个事物使用另一个事物时使用依赖关系，大多数情况下依赖关系体现在某个类的方法使用另一个类的对象作为参数。

在UML中，依赖关系用带箭头的虚线表示，由依赖的一方指向被依赖的一方。

比如，驾驶员（Driver）开车，Driver类的drive()方法将车（Car）的对象作为一个参数传递，以便在drive()方法中能够调用car的move()方法，且驾驶员的drive()方法依赖车的move()方法，因此也可以说Driver依赖Car，Java代码为：

```
public interface Car
{
    public void move();
}

public class Driver
{
    public void drive(Car car)
    {
        car.move();
    }
}
```

其UML的画法为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gst7tzxlxgj30j2058t8x.jpg)

依赖关系通常通过三种方式来实现：

- 1. 将一个类的对象作为另一个类中方法的参数
- 2. 在一个类的方法中将另一个类的对象作为其对象的局部变量
- 3. 在一个类的方法中调用另一个类的静态方法

### 9. 关联关系、聚合关系、组合关系之间的区别

从上文可以看出，关联关系、聚合关系和组合关系三者之间比较相似，本文的最后就来总结一下这三者之间的区别。

关联和聚合的区别主要在于语义上：关联的两个对象之间一般是平等的，聚合则一般是不平等的。

聚合和组合的区别则在语义和实现上都有差别：组合的两个对象之间生命周期有很大的关联，被组合的对象在组合对象创建的同时或者创建之后创建，在组合对象销毁之前销毁，一般来说被组合对象不能脱离组合对象独立存在，而且也只能属于一个组合对象；聚合则不一样，被聚合的对象可以属于多个聚合对象。

再举例子来说：

- 你和你的朋友属于关联关系（Association），因为你和你的朋友之间的关系是平等的，关联关系只是表示一下两个对象之间的一种简单的联系而已，就像我有一个朋友
- 你和你借的书属于聚合关系（Aggregation），第一是因为书可以独立存在，第二是因为书不仅仅属于你，也可以属于别人，只是暂时你拥有
- 你和你的心脏属于组合关系（Composition），因为你的心脏只是属于你的，不能脱离与你而存在


#### 本文第三部分转载自文章：

- [UML类图](https://www.cnblogs.com/xrq730/p/5527115.html)

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)