---
layout: post
title: 一起来踩配置 Disqus 的坑
categories: Jekyll
description: 如何在博客中配置Disqus
keywords: Disqus, Jekyll
---

搭好新博客的骨架也有几天了，可是留言区一直在报错。老博客也尝试过配置，不过那个时候也是不停报错，导致我一怒之下取消了评论区。这回为了彰显逼格，可不能再这么粗暴了╮(╯_╰)╭

下面从注册完 Disqus 账号开始记录一下整个配置流程中的易错点。

## 在 Disqus 中新建

### Create New Site

在 [Disqus](https://disqus.com) 首页上选择 `Get Started` ， 然后在新页面中选择 `I want to install Disqus on my site` ，进入 `Create a new site` 页面。

这里要注意两点：

1. 页面叫 Create a new site ，但是事实上不会真的新建一个网站。只是给你指定的网站建立一个项目，该网站所有的评论通知会 集中在这个项目里；
2. `Website Name` 是你之后要插入到 `_config.yml` 中的名字，而**非用户名**。这个名字也并非不能修改，之后到 `Setting - General` 中可以修改。 

### Configure Disqus for Your Site

顺着一路下来就会来到 `Configure Disqus for Your Site` 页面，这里设置也要注意一点：

1. `Website URL` 要输入完整的地址，比如如果像我一样使用的是 GitHub Pages ，就要加上 `https://` 的协议名；

配置完信息之后，在 Disqus 网站上的新建+设置部分就完成了。下面讲一下在自己网站上的配置。

## 在自己网站上配置 Disqus 信息

因为本博是采用 Jekyll 框架，并且是 Fork 自现有主题，所以这里不会重新介绍从零开始的配置，只简单讲一下需要加入/修改哪些文件和信息。

### 在 _config.yml 中配置

在 `_config.yml` 中加入如下的配置信息：

```
comments_provider: disqus
disqus_username: %WEBSITE NAME%
```

`disqus_username` 其实是上面提到的 `Website Name` ，而非你在 Disqus 上面注册的用户名。 

p.s 觉得误导可以改名，po主在这里就懒得改了。

### 设置 comments.html 页面

因为我这个主题是 Fork 而来，原 REPO 已经涵盖了此页面，主要注意 Disqus 的地址对不对：

```
var shortname = '{ { site.disqus_username } }';
s.src = '//' + shortname + '.disqus.com/embed.js';
```

因为这里没有采取写死 URL 的方式，才有上面需要设置 disqus_username ；如果你想写死这里也没有关系，上面的设置可以删去 `disque_username` 。但我觉得原 REPO 写的特别好的地方就是这里，不去写死——后来者需要修改的话只用改 _config.yml 就可以了，无需修改这些 HTML 页面。

## 发表评论&审核

按照 Disqus 的政策：

> Note: Registered users must now verify their email address prior to posting a comment. Pre-moderation is always enabled for guest comments.

- 注册用户需要认证 email ，再根据 Site Owner 的设置看评论是否要审核才能发表；
- 匿名用户是否可以发表评论需要看 Site Owner 的设置，而且必须要通过审核才能发表。

吐槽一句，其实这样真的不方便，毕竟很多急于求助的同学可能觉得注册流程很麻烦。但是想想我之前被垃圾评论攻击过的 Sina SAE ……好吧你们还是注册吧╮(╯_╰)╭

我还是那一句，认真求助的同学提的问题我一定会尽可能详细的回复——我自己也是小菜鸟，也希望我求助的时候大神们会耐心的解答我，所以我会按照我所期望被对待的样子去对待别人。
