---
layout: post
title: 【学习记录】 xlwings 学习笔记·第一弹
categories: [xlwings, excel]
description: xlwings 学习笔记
keywords: xlwings, excel
---

## 0. 一些背景故事

作为一个 HR ，日常工作接触最多的除了 Word 就是 Excel ，在公司没有良好的软硬件支持的情况下，可能会存在大量的手工账。这些手工活通常是重复性极强的机械式工作，比如核对考勤。

作为一个~~能摸鱼就摸鱼的~~打工人，最讨厌的就是这种无意义且累人的活儿，毕竟，有这些时间用来看看逼乎也好啊，还有大量未完结的文等待我们尊贵的盐选会员来临幸（呵

因此，在 2023 年伊始，信女发下要改变事事手工账窘况的宏愿，希望今年可以有效提升工作的自动化程度。人生苦短，不如Python，Let's xlwings.

## 1. 配置

xlwings 配置其实有两部分，一部分是在 Excel 里装插件，一部分就是常规性的安装库。如果不需要和 VBA 有什么联动（即要不要直接在 Excel 中通过控件来唤起），可以省略在 Excel 里装插件的步骤。

1. 安装 xlwings 库（我这里用 Anaconda 来管理，此前为处理 Excel 有关的工作专门创建了一个环境）： `conda install xlwings`
2. 在 Excel 中安装插件： 具体步骤可以参考 [Python xlwings操作Excel（摸鱼划水必备技能）——（2）python xlwings与VBA间的互相调用](https://blog.csdn.net/m0_59160272/article/details/125948932)

如果需要通过 VBA 唤起，建议使用 `xlwings quickstart filename` 直接创建对应 py 和 xlsm 文件。

## 2. 一些开发过程中的体会和提示

xlwings 使用起来其实更像一个基础的桥梁，让你能够通过代码去操作 Excel 文件，比如增删改查行列单元格这些基础操作，但 xlwings 不是一个数据分析的工具，是没有办法来替你做一些超过 Excel 本身功能的事情的。处理逻辑也更基础、碎片化，并不是像处理数据库一样可以直接 `select xx from table`，把整个 Excel 表想象成一个大数组可能更为合适。

### 2.1 一些踩坑的记录

1. 判断整个表共有多少行列：行数可以用`sheet.used_range.shape[0]`，列数可以用`sheet.used_range.shape[1]`
	- 但要注意整个表格行列是从 (0,0) 开始算的
2. 删除整行：如果是批量要删除，需要倒序来删，因为删除整行时行号会变动。
	- 删除行：`sheet.range(row, col).api.EntireRow().Delete()`
3. 读取单元格内容：`sheet[row, col].value`
	- 一定别忘了 `.value`，单元格内容也不是默认 string 类型，如果要处理字符串，要记得通过 `str()` 来强制转换
4. 日期处理：`datetime.datetime.strptime(strDate, '%Y-%m-%d').date()`
	- 要注意日期内容，有些并不是标准的日期形式，比如“2023-03-15 周三”这样，要先处理成标准日期形式
	- 注意`'%Y-%m-%d'`大小写，会代表不同的内容
	- 如何取当前月份最后一天：`datetime.date(year, mon, calendar.monthrange(year, mon)[-1])`
5. 单元格背景和字体颜色设置：参考[python xlwings中颜色值](https://zhuanlan.zhihu.com/p/538263850)
	- 设置单元格背景色时，可以用 RGB 和 0x 开头的 hex , 设置字体颜色时，用api接口，此时的 fontcolor 不能用 RGB ，只能用 0x 开头的hex
	- 且跟普通转换得到 hex 值会有区别（字符对掉了位置）

### 2.2 一些奇怪的问题

1. 发现在 Excel 中使用控件唤起时灵时不灵，跑样例可以，但跑我自己的写的代码就会卡住。初步判断是因为我开发的程序需要输入，但是控件唤起时未能激活控制台输入？
	- TODO List +1

## 3. 未完结语

xlwings 应该会成为我未来一段时间的自动化方案里的有效助力，这个系列也将持续更新……吧。（浅浅的 FLAG
