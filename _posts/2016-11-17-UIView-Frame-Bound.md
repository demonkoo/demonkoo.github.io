---
layout: post
title: UIView 中的 frame 与 bound
categories: iOS
description: UIView 中的 frame 和 bound 区别
keywords: iOS, UIView, frame, bound
---

frame & bound 的区别应该是 iOS 开发面试必考题之一，今天详细解析一下。

p.s 今天只写基础 UIView 的 frame & bound ，涉及 scroll view 的内容下次积到 scroll view 专题中写。（突然想起 Lancy 以前给我讲 scroll view 的情景，怀念题库(ಥ_ಥ)）

## Frame

### 定义

表示一个 view **在它的 super view 坐标系**中的位置。

这里注意两个容易疏忽的问题：

1. **不是所有 view 的原点都在屏幕左上角**；
2. 当前 view 的 frame 是它在 superview 中的位置，**为其添加 subview 的时候要特别注意**。
	- 即如果使用 self.curView.frame 来初始化 curView 的 subview ，此时其位置会是根据 curView 的 superview 而定。

上两张图对比一下帮助理解：

- redView 的 superview 就是 rootView （rootView 是覆盖住整个屏幕的）；
- greenView 是 redView 的 subview 。

<img src="/images/post/2016-11-17-UIView-Frame-Bound/frame-redView.jpg" width="30%"/>
<img src="/images/post/2016-11-17-UIView-Frame-Bound/frame-greenView.jpg" width="25%"/>

### 修改 frame 的 width & height

相当于**按住 view 的左上角，把 view 的右下角移动指定的距离**，跟我们直觉上的缩放方式一致。

## Bound

### 定义

表示以自身 view 原点（默认为 view 左上角）为基准的坐标系。

同样要注意一个问题—— **view 的原点可以通过设置 origin 而变化**，也即 view 的左上角坐标可以不从 0 开始。

### 修改 bound 的 width & height

相当于**按住 view 的 center 保持不变让 view 向四周扩散**，而 center 是一个 CGPoint 对象，**表示一个 view 在其 super view 坐标系中**，中心点的位置。

### 修改 bound 的 origin

相当于我们给一个 view 的 bounds 坐标系的原点设置了一个新的基准。

例如：

```swift
self.redView?.bounds.origin.x = 10
self.redView?.bounds.origin.y = 10
```

当我们把 bounds.origin 修改为 (10, 10) ，就相当于告诉 iOS ，这个 view 左上角的坐标不再从 0 开始，而是从 (10, 10) 开始。

### bound 和 frame 的联系？

当一个 view 拥有 sub view 时，它的 bounds 就是 sub view 的 frame 坐标空间。

如上面的例子，如果此时有一个 greenView 的 frame.orgin 为 (10, 10) ，则此时他们的左上角会重合，像这样：

<img src="/images/post/2016-11-17-UIView-Frame-Bound/bound-origin.jpg" width="25%"/>

## Bound v.s Frame

来看一个例子：

假设我们要把 greenView 内嵌在 redView 里，让它到 redView 四边的距离都是 10 ，即如下图：

<img src="/images/post/2016-11-17-UIView-Frame-Bound/bound-frame-example.jpg" width="25%"/>

那么应该如何设置这个 greenView 的 frame 呢？

### insetBy(dx:dy:)

首先提供一个可以方便裁剪矩形大小的函数 `insetBy(dx:dy:)` ，其具体作用是可以帮我们根据一个已有的矩形和 x/y 轴的差值创建新的矩形，也即裁切 dx & dy 的宽度。

#### 用 frame ?

如果我们这么写：

```swift
self.greenView = makeMyView(
    frame: self.redView!.frame.insetBy(dx: 10, dy: 10),
    bgColor: UIColor.green)

self.redView?.addSubview(self.greenView!)
```

会发现 greenView 的坐标是相对于 redView 左上角的。但是 self.redView!.frame 的值却是相对于屏幕左上角的，因此把这个矩形裁掉 10 ，放在 redView 里，一定不会是我们想要的结果，这也是我在上面 Frame 中强调过的“当前 view 的 frame 是它在 superview 中的位置，**为其添加 subview 的时候要特别注意**”。

#### 用 bound !

实际上，我们需要的是，以 redView 左上角为基准的 redView 自身的大小。然后，根据这个矩形去计算 greenView 的大小。所以，正确写法应该是：

```swift
self.greenView = makeMyView(
    frame: self.redView!.bounds.insetBy(dx: 10, dy: 10),
    bgColor: UIColor.green)

self.redView?.addSubview(self.greenView!)
```

## 不同 view 中坐标系的转换

### from view → 自身

把 from view 坐标系中的 CGRect / CGPoint 转换成方法调用者坐标系中的值：

- convert(_ point: CGPoint, from view: UIView?);
- convert(_ rect: CGRect, from view: UIView?);

#### 例子

当 redView 和 greenView center 重合时， greenView?.center 可以这样算出来：

```swift
let greenViewCenter = self.redView?.convert(
    self.redView!.center,
    from: self.redView!.superview)
```

因为 greeView 的 center 和 redView 重合，也即求 redView 的 center 在**自身坐标系**中的位置，因此只要把 redView 的 center 从其 superview 转换到自身坐标系即可。

### 自身 → to view

把方法调用者坐标系中的 CGRect 和 CGPoint 转换成 to view 坐标系中的值：

- convert(_ point: CGPoint, to view: UIView?);
- convert(_ rect: CGRect, to view: UIView?);

