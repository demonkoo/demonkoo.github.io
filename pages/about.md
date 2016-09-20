---
layout: page
title: About
description: 不要怂，就是干
keywords: Koo, Lilin Gu, 小古
comments: true
menu: 关于
permalink: /about/
---

是一个不像码农的程序员。

还有什么比做一个有趣的人更有趣？

## 联系

* GitHub：[@demonkoo](https://github.com/demonkoo)
* 博客：[{{ site.title }}]({{ site.url }})
* 微博: [@某古在小清新路上一骑绝尘而去](http://www.weibo.com/demonkoo)
* 知乎: [@呆萌古](https://www.zhihu.com/people/demon-koo)
* 豆瓣: [@燃烧吧小宇宙](https://www.douban.com/people/64274502/)

## Skill Keywords

#### Software Engineer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_software_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Mobile Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_mobile_app_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Windows Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_windows_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>
