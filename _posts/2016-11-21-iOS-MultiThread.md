---
layout: post
title: iOS 多线程初见
categories: iOS
description: 介绍基础的 iOS 多线程编程
keywords: iOS, multi-thread
---

以前在题库的时候也写过一点点关于多线程的东西，不过充其量就是抄一下前人写的回调，改改拿来用。但是关于多线程这方面的东西整个还是相当模糊混乱（少壮不努力的后果(ಥ_ಥ)），今天就来仔细梳理一下。

为什么要用多线程这里不多说了，下面主要用下载图片作为主要例子来分析。

## Serial Queue

### Serial Queue v.s Main Queue

一个 serial queue 和 Main Queue 使用的队列是非常类似的：

- 它们都只能顺序执行队列中的任务；
- main queue 可以和多个 serial queue 并行执行；

### Create & Async

```swift
// Create a serial queue
let serialQueue = dispatch_queue_create("images", DISPATCH_QUEUE_SERIAL)

dispatch_async(serialQueue, {
    let img = Downloader.downloadImageWithURL(self.imageUrls[0])
    dispatch_async(dispatch_get_main_queue(), {
        self.imageView.image = img
        self.imageView.clipsToBounds = true
    })

    // Omitted for simplicity...
})
```

需要注意**所有更新 UI 的操作，必须在主线程中完成**，因此要把更新 UI 的操作切回到 main queue 中。

### 多个 serial queue

- **单个** serial queue **内部**任务是**串行**执行的；
- **多个** serial queue **之间**是**并行**执行的


## Concurrent Queue

### Global Concurrent Queue

按优先级排序为：

1. DISPATCH_QUEUE_PRIORITY_HIGH
2. DISPATCH_QUEUE_PRIORITY_DEFAULT
3. DISPATCH_QUEUE_PRIORITY_LOW
4. DISPATCH_QUEUE_PRIORITY_BACKGROUND

其中，高优先级队列中的任务，会先于低优先级队列中的任务被执行

### Concurrent Queue v.s Serial Queue

作为一个队列，concurrent queue 中的任务也按照进入队列的顺序“启动”，但是和 serial queue 不同，它们不用等待之前的任务完成，iOS 会**根据系统资源的情况启动多个线程并行执行**队列中的任务。

### Create & Async

```swift
var currQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)

dispatch_async(currQueue, {
    let img = Downloader.downloadImageWithURL(self.imageUrls[0])
    dispatch_async(dispatch_get_main_queue(), {
        self.imageView.image = img
        self.imageView.clipsToBounds = true
    })
})
```

#### 自定义 Concurrent Queue

```
let currQueue = dispatch_queue_create("images", DISPATCH_QUEUE_CONCURRENT)
```

## Async v.s Sync

引自[iOS-多线程编程学习之GCD——串行队列和并发队列(五)](http://lysongzi.com/2016/02/26/iOS-多线程编程学习之GCD——串行队列和并发队列-五/)，原文有示例，建议大家看看，我这里把精华结论抄过来：

> 当我们创建了队列之后，我们需要把任务添加到队列中，并指定以同步还是异步的方式执行添加到队列中的任务。
  
其区别在于：

- 同步，其会在**当前线程**立即执行添加的任务（无论是串行还是并行队列都如此）。
- 异步，其会新创建一个**新的线程**来执行任务。而异步对于串行和并行队列的又不一样的意义的。
- 新添加的任务都会放到新创建的线程，即**每个任务在单独线程中**，并且所有的任务都是**并发执行**的。

## Operation Queue

**Serial & concurrent queue，在iOS中叫做 Grand Central Dispatch (GCD)**。尽管 GCD 对线程管理进行了封装，如果我们要管理队列中的任务（例如：查看任务状态、取消任务、控制任务之间的执行顺序等）仍然不是很方便。为此，iOS 基于 GCD 对多线程任务进行了进一步封装，提供了一个面向对象方式的多任务执行机制，叫 Operation Queue 。

### NSOperationQueue

- 队列中的任务会**并行**执行
- **不遵从 FIFO** 的原则
- 不是简单的 closure ，而是被封装成了一个 NSOperation 类

### NSOperation

NSOperation 是一个抽象类，不能直接生成对象，iOS 有两个基于 NSOperation 实现了两个具象类：

- NSBlockOperation - 我们可以把它理解为是一个 closure 的封装
- NSInvocationOperation - 在 Objective-C 中通过 selector 指定要调用的任务
	- 但由于使用 NSInvocationOperation 可能带来的类型安全以及 ARC 安全的问题，Apple 从 iOS 8.1 开始去掉了这个类。

### 创建队列 & 添加任务

#### 直接添加 closure

```swift
        var queue = NSOperationQueue()

        queue.addOperationWithBlock({
            let img = 
            	Downloader.downloadImageWithURL(self.imageUrls[0])
            
            NSOperationQueue.mainQueue().addOperationWithBlock({
                self.imageView.image = img
                self.imageView.clipsToBounds = true
            })
        })
```

#### 添加 NSBlockOperation 对象

```swift
        // 1. Create a NSBlockOperation object
        let op = NSBlockOperation(block: {
            let img = Downloader.downloadImageWithURL(self.imageUrls[0])
            
            NSOperationQueue.mainQueue().addOperationWithBlock({
                self.imageView.image = img
                self.imageView.clipsToBounds = true
            })
        })
        
        // 2. Set finish callback
        op2.completionBlock = { print("image downloaded") }

        // 3. Add to operation queue manually
        queue.addOperation(op)
```

添加 NSBlockOperation 的好处是可以添加回调函数等，操作更灵活。

### 设置任务间的关联性

添加关联性，会影响**执行顺序**&**任务打包**（取消任务时会将关联任务一起取消）。

#### 添加关联性

如有 1 2 3 三个任务，希望按 3 2 1 的顺序执行，那么关联性应该这样设置：

```swift
// 先设置关联性，再添加到队列中
op2.addDependency(op3)
op1.addDependency(op2)

queue.addOperation(op3)
queue.addOperation(op2)
queue.addOperation(op1)
``` 

#### 取消关联任务

取消关联任务时会有如下规则：

- 对所有已经完成的任务，取消操作不会有任何结果
- 如果一个任务被取消，所有和它关联的任务也会被取消
- 任务被取消之后，**completionBlock 仍旧会被执行**

取消状态可以通过访问 NSBlockOperation 对象的 cancelled 属性查询。