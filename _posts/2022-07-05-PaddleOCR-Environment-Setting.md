---
layout: post
title: 【整点活吧】从零开始开发干审表信息提取工具——基础环境配置
categories: [Python, Anaconda]
description: 搭建 PaddleOCR 基础环境
keywords: Anaconda, PaddleOCR
---

## 0. 故事前传

许久没有来记录些什么了，上一篇发表时间还是 2016 年的事情，一晃数年，如今的我也不再干开发了——现在是个人人喊打的 HR 。虽然离开开发岗位许久，但对整活的热情不增反减，小活这几年也整过不少，可能不靠这个吃饭后，~~用爱发电~~**有技巧的摸鱼**最开心吧┓( ´∀` )┏

本次需求源于国企人力岗位避不掉的一个东西——干部任免审批表，要求是把其中指定信息批量抽取填入到 Excel 表中。

正常情况来说如果是一个 Word 文件用 VBA 或者 Python 直接进行提取无疑是更好的选择，可是用总部的大聪明系统导出的干审表 PDF 转码后，发现里面文字信息竟然被处理成了单独的一条条小图片~~听我说谢谢你~~，也就是说并不能直接复制粘贴到 Excel 中，必须通过进一步的处理，把图片处理成文字后再进行抽取——所以整件事实际上就变成了 OCR 检测识别。

最初的想法是基于我之前写过的简历信息提取插件（基于 ResumeSDK 开发）来改良，但改完实践之后发现 ResumeSDK 针对的还是普通简历处理，所以“太有自己的想法”，云端引擎会对识别检测的信息进行处理后再返回结果，而干审表实际已经是个制式表，我只是要指定区域 100% 纯检测识别结果即可，所以其实用原插件改良并不合适。

逛知乎的过程中看到**虾米小馄饨**大佬写的[用 Python 写一个图像文字识别 OCR 工具](https://blog.csdn.net/Bit_Coders/article/details/121561632)，发现跟我的需求贴合极高，于是就进了坑。可惜大佬主要还是介绍的整体思路，过程中各种小坑不断，配环境都能配出 PTSD ……不废话了，今天先讲一下配环境和里面可能存在各种的坑。


## 1.运行环境准备

本次主要是基于 [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) 进行开发，一切环境配置要求都要严格按照官方推荐的流程和要求来，否则问题更多（别问我怎么知道的o(╥﹏╥)o）。

我在公司 PC 上配的环境，设备是 Windows 10 + 集显。我强烈建议如果之前**没有装过/用过 Anaconda 的朋友务必不要跳过**此步，不要觉得自己很牛逼，放下你的身段！不然之后要吃瘪的！（当然如果很熟悉这块的大佬，出现问题会自己搞定的，就当我没有讲过这个话……）

### 1.1.安装 Anaconda

步骤参照[官方指南](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/environment.md)。

主要的坑有下面几个：

1. 一定要用**管理员身份**运行安装包（安装包右键-以管理员身份运行），否则安装虽然显示完成，但会莫名缺失很大一块内容，可能连 Scripts 文件夹都没有，导致后续有问题。
2. 安装过程中勾选了 Add Anaconda3 to my PATH environment variable ，但实际可能并没有成功。在系统变量（此电脑右键-属性-高级系统设置-环境变量-系统变量）里检查一下，如果发现没有，添加如下路径（比如我的装在了E盘）：
```
E:\ProgramData\Anaconda3\Scripts
```
添加完 Path 之后记得要重启才会真的生效。

### 1.2.创建 conda 环境

官方指南其实前后有矛盾的，文章一开始推荐配置是 Python 3.7 ，实际流程中又按照 Python 3.8 创建的环境，我按 Python 3.8 配的没问题。

启动 Anaconda Prompt 时也要用**管理员身份**运行，否则后续安装各种框架的过程中有可能遇到奇怪的 Permission 问题。

## 2.安装框架

### 2.1.安装 PaddlePaddle

PaddlePaddle 是基础框架，一定要装的，不要和 PaddleOCR 安装搞混了， PaddlePaddle 安装步骤也参考[官方文档](https://www.paddlepaddle.org.cn/install/quick?docurl=/documentation/docs/zh/install/pip/windows-pip.html)。

注意此步安装要在上一步创建的 conda 环境中进行。

### 2.2.验证 PaddlePaddle

如果你使用的是 Pypi 源，这一步本来是没有坑的，但如果使用的是国内源，在验证环节：
```
import paddle
```
后就会发现报 protobuf 版本过高的错误。[错误原因](https://github.com/PaddlePaddle/Paddle/issues/43052)其实就是安装 Paddle 过程中会升级 protobuf 至4.21，降级回 3.20.1 就行：

```
pip install protobuf==3.20.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 2.3.安装 PaddleOCR

[官方文档](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/quickstart.md)里面只有一句话，实际坑集中在这里爆发。

如果之前没有按照官方文档使用 Anaconda 在创建 conda 环境一步步来弄的话，这一步直接在 Terminal 用 Pip 会有 Error 装不上。就算按照官方提示安装 shapely 安装包也不行，我试着用 conda 来安装 shapely 再装 PaddleOCR 也不行。最后东拼西凑强行解决了安装中的报错问题后，再次导入 PaddlePaddle 框架居然发现 Numpy 也莫名发生了问题……

所以之前没有老实按照官方指南来做，这一步又发生乱七八糟问题不会 debug 的朋友，还是从头一步步弄吧，血泪教训。

另，Windows 下可以用 conda 安装 shapely 包先，然后再安装 PaddleOCR ，这样就不会报错：

```
conda install shapely
```

### 2.4.测试 PaddleOCR

#### 2.4.1.代理问题

到这步其实大部分人都应该没有问题了，但如果很不幸的是你跟我一样在公司网络下配置，又恰好贵司代理服务器不允许/不支持 https 代理的话，会发现运行测试命令：
```
paddleocr --image_dir ./imgs/11.jpg --use_angle_cls true --use_gpu false
```
根本进行不下去了，会报 ProxyError 的问题。

其实这个问题在前面安装框架过程也可能发生，但可以通过-i + --trust-host 来解决，例如：
```
pip install openpyxl -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
```

但这里是 PaddleOCR 发起的，无法再像上面这样换成 http 来解决，公司电脑也不允许随便删除变动代理设置，这个时候该怎么办呢？

针对这个问题我查了非常多的资料，试了挺多办法都没有解决。最后在 **DavyCloud** 大佬这篇[Python 遭遇 ProxyError 问题记录](https://zhuanlan.zhihu.com/p/350015032)中终于找到了解决方案，我是用其中第1种方法解决的，即在用户变量中添加 HTTP_PROXY 和 HTTPS_PROXY 并全部设置为 http 的代理服务器地址来覆盖系统代理配置。

此时再用
```
import urllib.request;
print(urllib.request.getproxies())
```
检查，就会发现返回 https 代理已经变化了。

#### 2.4.2.OMP Error #15

当终于解决代理问题，准备美滋滋等待结果的时候，发现执行测试命令还在报错：

> OMP: Error #15: Initializing libiomp5md.dll, but found libiomp5md.dll already initialized.

这个问题在 **Chen John** 大佬[关于 OMP: Error #15: Initializing libiomp5md.dll, but found libiomp5md.dll already initialized. 错误解决方法](https://zhuanlan.zhihu.com/p/371649016)里面也有详解，主要就是因为 torch 和 Anaconda 有重复文件冲突，通过删除当前 env 下的 libiomp5md.dll 即可解决。

## 3.结语

万里长征第一步终于结束了，当然我自己踩的坑和 debug 过程远比上面写的多。

为什么很想把这件事做好，除了本身工作有需求，开发这样一个工具可以提高效率以外，也是想响应帅姐说的，摆烂了大半年，下半年得要有点进步了，哪怕是一点点呢。

做 HR 里面最会写代码的人，也挺好的。