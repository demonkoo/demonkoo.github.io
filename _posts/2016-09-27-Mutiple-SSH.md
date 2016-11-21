---
layout: post
title: 多 SSH 与 GitHub 账户管理
categories: SSH
description: 如何将不同的 SSH 绑定到对应的 GitHub 账户
keywords: SSH, GitHub
---

这几天在帮车王搭独立博客，因为都是用 GitHub Pages 来做，所以也分别建立了不同的账号。由于不想混淆两个人的信息（虽然都是我搭……），所以我想用两个不同的 SSH Key 来绑定对应的账号。

## 配置步骤

### 生成 SSH Key 并添加到 GitHub

进入 `~/.ssh` 后，生成第2个 Key ：

```
ssh-keygen -t rsa -C "车王的 email "
```

然后取公钥添加到 GitHub 上。

### 配置 SSH Config

因为之前生成过 config 文件，直接打开添加修改：

```
Host demonkoo.github.io
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/我的SSH Key
Host yuanshanlee.github.io
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/车王的SSH Key
```

记住两个 Host 的名字，在这里就是 `demonkoo.github.io` 和 `yuanshanlee.github.io`。

#### 测试连接

分别测试两个 SSH ：

```
$ ssh -T git@demonkoo.github.io
```

```
$ ssh -T git@yuanshanlee.github.io
```

看是否成功。

### 配置仓库

因为我已经把远程仓库拉取下来了，所以需要修改车王博客仓库的 `.git/config` 文件：

```
[remote "origin"]
  url = git@yuanshanlee.github.io:yuanshanlee/yuanshanlee.github.io.git
```

这里照着这种格式改就行，之前的 https 链接可以删去了。

## 结尾

以上就是如何建立两个不同的 SSH 并绑定到对应的 GitHub 账号以方便代码管理。

其实我觉得直接用自己的账号成为 Contributor 也不是不行……算了开心就好，我也学到了一点新知识（虽然我觉得以后也用不到╮(╯_╰)╭），今天就这样吧~

---

2016.11.21 Update

在设置 `~/.ssh/config` 文件的时候，最好保留一下 `Host github.com` 的设置：

```
Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/我的SSH Key
```

不然之后每次新建 github repository 的时候都得手动修改一下 url ，因为默认提供的形式会是 `git@github.com:demonkoo/XXX.git` 这种形式，如未保留，则需修改为 `git@demonkoo.github.io:demonkoo/XXX.git` 。

还是建议保留，不然看起来十分别扭╮(╯_╰)╭如果这里你用新的 SSH Key 记得去自己的 GitHub Setting 中添加。