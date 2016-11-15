---
layout: post
title: C# & .NET 中的委托和事件
categories: [C#, .NET]
description: 讲解C#中委托的原理和事件的演变
keywords: Delegate, Event, C#, .NET
---

首先声明，本文的技术讲解部分提炼总结自：[C# 中的委托和事件](http://www.tracefact.net/CSharp-Programming/Delegates-and-Events-in-CSharp.aspx) 强烈建议大家先看看这篇，讲的非常透彻，我这篇权当是精简梳理，便于我自己和各位看官日后快速回顾。

## 问题起源

前段时间做开发的时候遇到一个抓马情况：

我做了一个“通过在下拉列表选择 Clearing House ID 后会自动在文本域显示其对应名称”的自定义控件。该控件**在通常情况下与其他控件无关联**，但在某个交易中，业务提出需要在选择完 Clearing House ID 之后，自动从数据库中取回其最近一笔记录的 Value Date 并显示到界面对应栏位里。

显然，最简单的方式自然是开放 combobox ，设置其为 public 之后就可以自行在这个特定交易中添加一个事件了。但是这样做会破坏其封装性，我并不希望别人可以随便更改我这个 combobox ，所以此时最好的方法就是增加一个事件代理——效果相当于在不影响 combobox 整体的 modifier 情况下，单独把选择事件开放。

接下来开始讲讲 C# 中的 Delegate 和 Event 的演化和联系。

## C# Delegate 的诞生

### 将方法作为方法的参数

#### 方法参数的类型？

方法参数类型定义应该能够确定方法参数可以代表的方法种类，再进一步讲，就是方法参数可以代表的**方法**的**参数类型**和**返回类型**。

为了表达上述的方法参数类型，Delegate 诞生了。

#### 方法参数 → 委托

> Delegate **定义了方法参数所能代表的方法的种类**，也就是方法参数的类型。

对比一下 Delegate 和方法签名，e.g:

```cs
// 方法签名
public void EnglishGreeting(string name);
// 委托
public delegate void GreetingDelegate(string name);
```

会发现其实对于方法签名来说，委托的区别不过是增加了一个 delegate 关键字。但是根本的关键是我们现在可以通过这个委托，传入对应的方法：

```cs
public void GreetPeople(string name, GreetingDelegate MakeGreeting){
    MakeGreeting(name);
}
```

> 委托在编译的时候会编译成类。**因为 Delegate 是一个类，所以在任何可以声明类的地方都可以声明委托**。

#### 小结

委托是一个**类**，它定义了**方法的类型**，使得可以将方法当作另一个方法的参数来进行传递，这种将方法动态地赋给参数的做法，可以避免在程序中大量使用 If-Else / Switch 语句，同时使得程序具有更好的可扩展性。

### 将方法绑定到委托

#### 绑定方法是委托的一个特性

> 可以将多个方法赋给同一个委托（用 `+=` ），或者叫将多个方法绑定到同一个委托，当调用这个委托的时候，将依次调用其所绑定的方法。

```cs
static void Main(string[] args) {
    GreetingDelegate delegate1;
    // 先给委托类型的变量赋值
    delegate1 = EnglishGreeting;
    // 给此委托变量再绑定一个方法
    delegate1 += ChineseGreeting;

    // 两种调用方式（将先后调用 EnglishGreeting 与 ChineseGreeting 方法）：
    // 1. 用原类调用
    GreetPeople("Jimmy Zhang", delegate1);  
    Console.ReadKey();
    
    // 2. 用委托调用
    delegate1 ("Jimmy Zhang");   
    Console.ReadKey();
}
```

注意这里第一次用的“=”，是赋值的语法；第二次，用的是“+=”，是绑定的语法。如果第一次就使用“+=”，将出现“使用了未赋值的局部变量”的编译错误。

#### 赋值 v.s 绑定

对比一下两种生成委托的方式，比较直观能反应问题：

```cs
// 可以编译运行
GreetingDelegate delegate1 = new GreetingDelegate(EnglishGreeting);
delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法

// 编译错误
GreetingDelegate delegate1 = new GreetingDelegate();
delegate1 += EnglishGreeting;   // 这次用的是 “+=”，绑定语法。
delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法  
```

#### 小结

使用委托**可以将多个方法绑定到同一个委托变量**，当**调用**此变量时(这里用“调用”这个词，是因为此变量代表一个方法)，可以**依次调用所有绑定的方法**。

## C# Event 的诞生

### 把 Delegate 封装进内部

根据面向对象设计，我们应该把委托也封装到类中，如：

```cs
public class GreetingManager{
    // 在 GreetingManager 类的内部声明 delegate1 变量
    public GreetingDelegate delegate1;  

    public void GreetPeople(string name) {
        // 如果有方法注册委托变量，通过委托调用方法
        if(delegate1!=null){     
          delegate1(name);
       }
    }
}
```

#### 存在问题？

1. 并不是所有的字段都应该声明成 public ，合适的做法是应该 public 的时候 public ，应该 private 的时候 private ；
	 - 如果把 delegate1 声明为 private 客户端对它根本就不可见，而声明委托的目的就是为了把它暴露在类的客户端进行方法的注册；
	- 如果把 delegate1 声明为 public ，在客户端可以对它进行随意的赋值等操作，严重破坏对象的封装性。
2. 第一个方法注册用“=”，是赋值语法，因为要进行实例化，第二个方法注册则用的是“+=”。但是，不管是赋值还是注册，都是将方法绑定到委托上，除了调用时先后顺序不同，再没有任何的分别，这样不是让人觉得很别扭么？

因此，我们亟需一个新概念解决以上的问题，这就是事件。

### Event 是封装委托类型的变量

Event 封装了委托类型的变量，使得在类的内部，不管你声明它是 public 还是 protected ，它**总是 private 的**。在类的外部，注册“+=”和注销“-=”的访问限定符与在声明事件时使用的访问符相同。

```cs
public class GreetingManager{
    // 声明一个事件
    public event GreetingDelegate MakeGreet;

    public void GreetPeople(string name) {
        MakeGreet(name);
    }
}
```

此时绑定方法不用像之前一样先赋值再绑定了：

```cs
static void Main(string[] args) {
    GreetingManager gm = new  GreetingManager();
    gm.MakeGreet += EnglishGreeting;
    gm.MakeGreet += ChineseGreeting;

    gm.GreetPeople("Jimmy Zhang");
}
```

#### 拓展：Observer 设计模式

有了上面的事件概念，现在我们可以有一个新的设计模式：通过事件来驱动不同对象之间的联系。

Observer设计模式中主要包括如下两类对象：

- Subject：监视对象，它往往包含着其他对象所感兴趣的内容；
- Observer：监视者，它监视 Subject ，当 Subject 中的某件事发生的时候，会告知 Observer ，而 Observer 则会采取相应的行动。

> Observer 设计模式是为了定义对象间的一种**一对多**的依赖关系，以便于当一个对象的状态改变时，其他依赖于它的对象会被自动告知并更新。Observer 模式是一种**松耦合**的设计模式。

## .NET 中的 Delegate & Event

.NET 中的委托与事件其实本质和上边是一致的，只是 .NET 在上面的基础上多了一些规范和 `EventArgs` 参数。

### 编码规范
- 委托类型的名称都应该以 EventHandler 结束；
- 委托的原型定义：有一个 void 返回值，并接受两个输入参数：
	- 一个 Object 类型：代表 Subject ，回调函数可以通过它访问事件触发对象，也即 Subject 本身；
	- 一个 EventArgs 类型/继承自 EventArgs 的类型：包含 Observer 感兴趣的数据。
- 事件的命名为委托去掉 EventHandler 之后剩余的部分。
- 继承自 EventArgs 的类型应该以 EventArgs 结尾。

### 大致框架

#### Subject 对象内部

- 声明委托
- 声明委托事件
- 自定义 EventArgs 衍生类
	- 内部属性即提供给 Observer 的内容
- 定义触发事件（此处会调用委托事件）
	- 建议添加 virtual 关键字，以便衍生类可以修改

```cs
// Omitted...
	   // 声明委托
       public delegate void BoiledEventHandler(Object sender, BoiledEventArgs e);
       //声明事件
       public event BoiledEventHandler Boiled; 
       
       // 定义 BoiledEventArgs 类，传递给 Observer 所感兴趣的信息
       public class BoiledEventArgs : EventArgs {
           public readonly int temperature;
           public BoiledEventArgs(int temperature) {
              this.temperature = temperature;
           }
       }

       // 可以供继承自 Heater 的类重写，以便继承类拒绝其他对象对它的监视
       protected virtual void OnBoiled(BoiledEventArgs e) {
           if (Boiled != null) { // 如果有对象注册
              Boiled(this, e);  // 调用所有注册对象的方法
           }
       }
       
       // 烧水
       public void BoilWater() {
       		// Omitted...
       		// 建立 BoiledEventArgs 对象。
                  BoiledEventArgs e = new BoiledEventArgs(temperature);
                  OnBoiled(e);  // 调用 OnBolied 方法
       }
```

#### Observer 对象内部

- 实现委托事件

#### 外部对象

- 给 Observer 对象绑定方法