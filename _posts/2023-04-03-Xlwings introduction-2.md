---
layout: post
title: 【学习记录】 xlwings 学习笔记·第二弹
categories: [xlwings, excel]
description: xlwings 学习笔记
keywords: xlwings, excel
---

## 0. 需求简述

1. 将汇总表每月数据按每行对应部门拆分成单独的子表
	- 每行根据单位格内容可能会对应子表中不同的列，并非固定一对一
2. 子表中原先可能存在内容，需要采用插入更新的方式
	- 本次指定为插入到“合计”行之上
	- 需要更新部分公式

## 1. 踩在自己的肩膀上——打包发布 Python 包

### 1.1 为什么需要打包发布

在上一次开发中已经编写了一些有用的函数方法，本次也会用到。当然最简单的方法是直接复制粘贴过来，不过这样堆代码的行为真的很不优雅，所以我决定将这些有用的函数打包，今后直接导入使用就行。我先前只有 C/C++ 编译发布库的经验，还没有过发布 Python 包的经验，也趁机学习一下。

主要步骤参考[如何发布自己的 python 包？](https://juejin.cn/post/6844903950550827022)，但有一些简化。

- 本次为首次尝试，还不成熟，所以没有建 Github 仓库，自然没有第 1, 2 步操作（不过后期还是会考虑放到 Github 上管理的），同时第 3 步 setup.py 中也无需添加 url 属性；
- 如果没有编写 README.md 的话，其实 setup.py 可以简化成这样的形式：

```
import setuptools


setuptools.setup(
    name="xxx",
    version="0.0.1",
    author="xxx",
    author_email="xxx",
    description="Utils function based on XLwings",
    install_requires=['xlwings'],
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)
```

但 long description 的内容是会显示在首页的，空白的话真的就是毛都没有，极丑。（别偷懒……

- 因为我的包主要就是一些函数方法，所以可以直接放在 __ init __.py 中实现。 以及发现并不是在 init 文件中 import 进来的包/模块就都会被导出，import 第三方库的话就没有被导出。  

### 1.2 一些安装方面的问题

因为我是用 Anaconda 来管理的，按上面步骤上传到 Pypi 之后，需要通过 PIP 来下载安装。此处需要注意的是，即便在 Anaconda 中切换到对应环境中来安装，最终 PIP 安装完的位置也不会像普通 `conda install XXX` 一样装到 `..\Anaconda3\envs\excel\Lib\site-packages` 中，而是会装到 `..\Anaconda3\Lib\site-packages` 中。

## 2. 一些开发过程中的记录

### 2.1 一些踩坑的记录

1. 通过 `sheet[row, col]` 方式访问单元格，行和列 index 都是从 **0** 开始计数的；但通过 `sheet.range(row, col)` 来访问的话，行和列 index 则都是从 **1** 开始计数的。
	- 普通获取单元格内容当然用前者，但是如果涉及希望填充公式（即会自动更新的）得用后者，`sheet.range(row, col).copy(原公式range)`
2. 注意上面自动填充的公式，如果是原公式是 `=SUM(某列中间几行)` 这样的形式，那在原选择区域下面插入列则不会自动更新公式，此时必须手动去更新公式内容。
	- 这里更新公式就要取单元格地址了，即 Excel 公式中的引用范围，比如 `A1:A10` 这样
	- 可以用 `sheet.range((左上角row, col), (右下角row, col)).get_address(False, False)` 这个方法来直接获取引用范围，再拼个 SUM 公式出来，赋值给 `sheet.range(row, col).formula` 就行
3. 插入行和跟上一篇的删除行正相反，要正序插入，因为插入行后面 index 都会变化。
	- 像本次这个需求，要求插入到合计行上面，其实完全可以只读一次合计行的 index 就行，本次批量固定插入到这个位置并不会错（因为全部都是当月数据），先插入数据会连着合计行一起自动往后调整。


## 3. 未完结语

中银要加钱！！！要特殊处理的地方也太多了，虽然整体难度不算大，但真的很烦！！！（被杨老师一杯酸奶骗了一个程序的某古碎碎念……