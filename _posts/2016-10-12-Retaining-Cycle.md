---
layout: post
title: Reference Cycle 出现 & 解决
categories: iOS
description: 循环引用问题的出现和解决
keywords: iOS, ARC, Swift
---

关于 Retaining Cycle，Swift 和 Objective-C 在问题发生和解决方式上本质相同，今天做一个状况分类和介绍对应的解决方式。

## 问题发生

由于 Swift 使用 ARC - Automatic Reference Counting 来管理内存，而 ARC 又是依靠 reference count 来管理`类对象`的生命周期（struct 和 enum 为`值类型`），所以当两个对象互相引用时就可能产生 Reference Cycle 问题。

> 和带有 Garbage Collection 的语言不同，当一个对象的 reference count 为0时， Swift 会立即删除该对象。

常见的问题场景分为两大类：两个类对象成员间互相引用和使用 Closure 时，下面会给出在这些场景下对应情况的解决方案。

## 解决方案

### 两个类对象间引用

#### 1. Class member 均允许为 nil

- 解决方案：**Weak reference**
	- 将其中一个 Class member 的引用标记为 weak 即可
- e.g: Apartment & Person
	- Apartment 可以没有 tenants ；Person 也可以没有 Apartment
	
```swift
class Person {
    let name: String
    var apartment: Apartment? 
}

class Apartment {
    let unit: String
    weak var tenant: Person?
}

```
	
##### Weak Reference

对于一个 weak reference 来说：

- 当其指向类对象时，并不会添加类对象的引用计数；
- 当其指向的类对象不存在时，ARC 会自动把 weak reference 设置为 nil ；

> 由于 weak reference 的特定，它只能被定义成 var 。

#### 2. Class member 只有一个允许为 nil

- 解决方案：**Unowned reference**
	- 将不允许为 nil 的 Class member 标记为 unowned 即可
- e.g: Apartment & Person
	- Apartment 必须有 owner ；Person 可以没有 Property
	
```swift
class Person {
    let name: String
    var property: Apartment?  
}

class Apartment {
    let unit: String
    unowned let owner: Person
}
```
	
##### Unowned Reference

和strong reference相比，unowned reference只有一个特别：不会引起对象引用计数的变化。

#### 3. Class member 均不允许为 nil

- 解决方案： **Unowned reference** + **Implicitly unwrapped optional**
- e.g: Country & City
	- Country 必须有一个 Capital；City 必须属于一个 Country
	

```swift
class City {
    let name: String
    unowned let country: Country
    
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}

class Country {
    let name: String
    // Implicitly Unwrapped Optional
    var capital: City! // default to nil
    
    init(name: String, capitalName: String) {
        self.name = name
        // PAY ATTENTION!
        self.capital = City(name: capitalName, country: self)
    }
}
```

调用时：

```swift
var cn: Country? = Country(name: "China", capitalName: "Beijing")
var bj: City? = City(name: "Beijing", country: cn!)
```

##### Implicitly unwrapped optional

上面的场景下，如果用一般的方式构建，则在 Country 的 init() 里，我们把正在创建的 Country 对象传递给了 City 的 init() 。

这样做只在语义上是正确的；语法上，Swift 认为构建 City 时，Country 还没有完成初始化（ self.captical 还没有确定的值），因此，它不允许我们在这个时候把 self 传递给 country ，这就陷入了一种“死循环”的困境。

因此要想构建 City 时，必须让 Swift 认为 Country 已经构造完来打破僵局，而唯一的做法就是 captical 有一个默认值 nil 。因此对于 Capital ，有了两个看似冲突的需求：

- 对 Country 的用户来说，不能让他们知道 capital 是一个 optional ；
- 对 Country 的设计者来说，它必须像 Optional 一样有一个默认的 nil 。

而解决这种冲突唯一的办法，就是把 capital 定义为一个 Implicitly Unwrapped Optional 。

此时问题又转化到了第 2 条的状况中，因此 City 的 country 属性也要标为 unowned 。

### 类对象与 Closure 间引用

当某个类中含有 closure ，并且此 closure 引用了这个类。

#### 解决方案

使用 **Capture List** ，e.g:

```swift
class HTMLElment {
    let name: String
    let text: String?
    // Pay Attention to 'Lazy'
    // Make sure name & text is initialized
    lazy var asHTML: Void -> String = {
    	// Capture list
    	[unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(self.text)</\(self.name)>"
        }
        else {
            return "<\(self.name)>"
        }
    }

    // Omit for simplicity...
}
```
有 2 点需要注意：

- lazy 可以确保一个成员只在类对象被完整初始化过之后，才能使用；
- 就算在 closure **内部多次使用 self** ，closure 对 self 的**捕获仅发生 1 次**（引用计数只加 1 ）。

#### Capture List

本质上来说，closure 作为一个引用类型，解决 reference cycle 的方式和解决类对象之间的 reference cycle 是一样的：

如果引起 reference cycle 的"捕获"不能为 nil ，就把它定义为 unowned ，否则定义为 weak 。而**指定“捕获”方式**的地方，叫做 closure 的 **capture list** 。

而实际在上面的例子中，捕获的 self 不能为 nil （不然 closure 本身也不存在），所以只能定义为 unowned 。

##### 语法要点

- 如果 closure 带有完整的类型描述，capture list 必须写在参数列表前面；
- 如果我们要在 capture list 里添加多个成员，用逗号把它们分隔开。

