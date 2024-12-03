---
layout: post
title: 【奇巧淫技】 iOS 和 Windows 间文件传输
categories: [iOS, iPhone, Windows, PC]
description: iOS 和 Windows 间文件传输
keywords: iOS, iPhone, Windows, PC
---

## 0. 一些背景&用户需求

今天这个分享来源于方老师的一个独特情况。

话说方老师新买了一台 MacBook （我不理解好端端的为什么要折磨自己……），她在 MacBook 上写了一份材料需要放到公司无网络的保密机中（即不可以使用任何联网方法，只能用便携式设备等进行文件传输），且由于 MacBook 只有 type-c 口，但她没有 type-c 口的 u 盘等设备，只能临时将文件存在 iPhone 中。同时，在 Mac OS 下，无法访问 iPhone 根目录，只能放在 Pages 文件夹里。

另，她在公司有另一台工作电脑，但基本只能访问公司局域网，外网访问限制。

## 1.解决方案

大致思路跟 [Windows 和 iOS 13无缝传输文件最快的方案](https://zhuanlan.zhihu.com/p/83983289) 和 [轻松实现iphone和windows传输文件](https://www.youtube.com/watch?v=NjKpO8K0GSo) 里一样（建议看后面这个视频教程，比文字讲的详细），即：

1. 在 PC 上设置共享文件夹，PC 作为服务器（注意 PC 要开启 smb 服务）
2. 通过 iPhone 文件 Apps 连接 PC 服务器，访问该共享文件夹

注：开启SMB服务教程 [Windows 之 win SMB(smb) 功能的开启设置和使用的简单说明](https://blog.csdn.net/u014361280/article/details/113678753)，当然视频里也有说明。

## 2.一些注意事项和坑

1. 先检查 PC 和 iPhone 是不是在同一网络下，尤其是如果 PC 是台式机/一体机时，极有可能是通过网线连接的，而不是 WiFi ，这样不行。
2. 设置共享文件夹权限时，实际没有必要开给 Everyone ，完全可以新建一个专门用户，权限设置给这个新用户，安全性更好一些。设置新用户的好处还有就是，这样你完全可以跳过教程里那些什么查找用户名，什么要加设备名怎么都弄不对之类的问题。
	- 创建新用户方法是在 控制面板-用户账户-管理其他账户-添加本地账户，注意要输入密码。
3. 没有第三了。

## 3.结语

其实这个教程没有什么屁用，因为像方老师这类不爱捣腾的用户，通常宁愿选择再掏钱买个新 u 盘（实际上她也是这么做的）。爱捣腾的用户，一般来说也轮不到需要看我这篇文章，所以……给自己留个操作记录吧。