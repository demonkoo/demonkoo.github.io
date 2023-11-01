---
layout: post
title: 【整点活吧】 Docker 安装踩坑记
categories: [Docker, SVM]
description: Docker 安装
keywords: Docker, SVM
---

## 0. 前情摘要

每逢年底 HR 必要面对的一件事就是：统计当前的薪酬情况。

本来这也是一个不算特别烦的活——前提是有薪酬管理系统的辅助。然而我司到目前为止仍是纯 Excel 手工账，且涉及两岸三地的人员发薪，统计起来就特别麻烦，稍有不慎就会出错。在本人发愤图强整理台账发现已经拓展到AY列以后，脑瓜子嗡嗡的……

大家都知道本人最讨厌的就是干机械性重复工作，所以我决定一定要整个薪酬系统。鉴于我司~~抠逼~~经费紧张问题，以及只有我一个人来开发的情况，找个开源的来改而不是自己重新造轮子显然是更好的选择。

比较了多款之后，最终觉得主要使用 Python 来开发的这款 [Frappe HR](https://github.com/frappe/hrms) 更符合我目前的现实情况。

## 1. Docker 安装

按照 Frappe HR 的部署要求，首先必须安装他们公司开发的两个底层模块，bench 和 ERPNext 。而这俩安装都是推荐用 Docker 的，所以第一步工作应该是安装 Docker 。

本来以为就是下载个安装包安装就可以了，结果坑接踵而至……

### 1.1 安装中

Docker 安装中防护软件可能会弹窗，注意要允许，否则会失败。

### 1.2 安装后

#### 1.2.1 运行报错

Docker 安装后运行，弹出错误提示：
> An unexpected error was encountered while executing a WSL command.Common causes include access rights issues, which occur after waking the computer or not being connected to your domain/active directory.
Please try shutting WSL down (wsl --shutdown) and/or rebooting your computer. If not sufficient, WSL may need to be reinstalled fully. As a last resort, try to uninstall/reinstall Docker Desktop. If the issue persists please collect diagnostics and submit an issue (Troubleshoot Docker Desktop | Docker Docs 371).

针对这个问题，我首先按照提示执行`wsl --shutdown`然后重启，无效；然后百度了一下，有说要安装 WSL 2 、还有说执行 `netsh winsock reset` 的等等，尝试均未成功。

最后还是在 Docker 官方论坛下面找到了答案：[An unexpected error was encountered while executing a WSL command](https://forums.docker.com/t/an-unexpected-error-was-encountered-while-executing-a-wsl-command/137525)

主要问题还是源于**没有开启虚拟化，这个要从 BIOS 来开启**。

#### 1.2.2 HHKB 问题（未解决）

我一直用的 HHKB 键盘，目前公司配的 HP Eliteone 一体机需要在开机时按 ESC 进入 BIOS 设置。

然而，不管是开机时按住 ESC 还是不停猛按 ESC ，都没法进入 BIOS 。我以为是 HHKB 有热键冲突的问题，还专门找了键盘测试工具试了一下，确认了 ESC 位置，然而就是不行。

最后只能从其他电脑拆了一个普通键盘过来才能终于能进 BIOS 设置界面了……

#### 1.2.3 BIOS 设置

各家设置页面有所不同，我司配的这台老掉牙机器连官网都没有对应指南如何开启 SVM (VTX)，最后在 YouTube 上面找到一个设置视频：

1. 按 F10 进入 BIOS Setup
2. 到 Adanced 页面，选择 System Options
3. 勾选 Virtualization Technology (VTx)
	- 下面那个 Virtualization Technology for Directed I/O(VTd) 不用勾选
4. 回到 Main 页面，选择 Save Changes and Exit。

重启之后 Docker 运行就没有再报错了。

## 3. 未完结语

说真的，贵司不单独给我配一台好机器真的说不过去。这才是第一步，路漫漫其修远兮……