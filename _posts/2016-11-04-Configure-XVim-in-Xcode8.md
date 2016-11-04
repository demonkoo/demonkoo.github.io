---
layout: post
title: 在 Xcode 8 中使用 XVim 
categories: Xcode
description: 在 Xcode 8 中配置 XVim
keywords: Xcode 8, Xvim
---

最近在看函数式编程和 Swift 3 相关的东西，想着既然都 Xcode 8 都上线了，我也来与时俱进一下吧——于是也没看各方反馈就直接升级了。

等了快仨小时，终于更新完毕——然后我发现——所有插件都用不了 _ (:з」∠) _

没有 XVim 用的我要死了，于是无奈之下开始了这场给 Xcode 8 配置插件的搏斗。

## Xcode 8 不可使用 Plugins 的原因

> Apple has allegedly disabled the use of plugins to avoid another incident like the Xcode Ghost 👻. In 2016's dub dub Apple announced their new Source Editor extensions. Unfortunately these extensions are still pretty limited, run completely isolated from the Xcode process and require user interaction to do anything. They may, however, solve your problem, and if they do just use them.

其实就是去年 Xcode Ghost 门导致各大厂商中标，果爹为了避免这个问题在 Xcode 8 中禁止使用插件。但是由于限制太多，导致原来插件可以做的事情现在做不到了。

## 解决方案

其他插件都好说，没 XVim 太要命，目前 XVim 已经给出一种解决方案：[Install XVim for Xcode 8](https://github.com/XVimProject/XVim/blob/master/INSTALL_Xcode8.md)

这其实不是对 XVim 做处理，而是去 re-codesign Xcode 8。但是我尚不清楚这样做和去除签名有什么区别。我希望尽量不要影响现存版本，所以我采用了另一种解决方案：

用 [MakeXcodeGr8Again](https://github.com/fpg1503/MakeXcodeGr8Again) 这个小程序来生成一个新的“破解”副本。

具体操作可以看这位童鞋的教程：[Xcode8使用插件方法以及乱打印没快捷注释的bug解决办法](http://www.jianshu.com/p/60f0e86fb97c)

## XVim 被 Playground 坑了

用上面的方法生成了一个 Gr 版本，打开会发现插件现在都可以用了。但是当你打开 Playground 的时候，你会发现一个新问题：XVim 在 Normal 模式中的光标不见了。

关于这个问题我 Google 了很久，最后找到这个 [No cursor in Playground #622](https://github.com/XVimProject/XVim/issues/622)

因为比较长所以我挑重点的说，这个光标问题不是 Xcode 8 第一次出现，之前在 Xcode 6 & 7 中已经存在。引起这个问题的原因是：

> The view used in Playground is called 'IDEPlaygroundTextView' that is dervied from DVTSourceTextView which is used when coding in ObjC, C++, etc. I do not know the reason but IDEPlaygroundTextView does not use intrinsic cursor drawing operation (which is inherited from NSTextView), which makes it unable to draw the cursor as we expect.

当时的解决方案是在 `~/.xvimrc` 中添加一句 `set blinkcursor` 可以解决，但是这个方案在 Xcode 8 中失效。

热心网友 @pebble8888 提供了一个可以使用的版本：[Click This](https://github.com/pebble8888/XVim/tree/develop) 

下载下来，cd 到目录，按照安装 XVim 的老方法 make 一下就可以使用了，亲测 Playground 中的光标没有问题。

### Detail...?

我看了一下这个小哥的代码，和 Playground Cursor 相关的是改动了 `XVim/DVTSourceTextView+XVim.m` 这个文件：

```
...Omitted...

- (BOOL)isIDEPlaygroundSourceTextView
{
    return [self isMemberOfClass:NSClassFromString(@"IDEPlaygroundTextView")];
}

// Drawing Caret
 - (void)xvim__drawInsertionPointInRect:(NSRect)aRect color:(NSColor*)aColor{
     // TRACE_LOG(@"%f %f %f %f", aRect.origin.x, aRect.origin.y, aRect.size.width, aRect.size.height);
 @@ -202,6 +207,17 @@ - (void)xvim__drawInsertionPointInRect:(NSRect)aRect color:(NSColor*)aColor{
             return [self xvim__drawInsertionPointInRect:aRect color:aColor];
         }
 
        if ([self isIDEPlaygroundSourceTextView]){
            // Playground
            NSGraphicsContext* context = [NSGraphicsContext currentContext];
            [context saveGraphicsState];
            NSUInteger glyphIndex = [self insertionPoint];
            NSRect glyphRect = [self xvim_boundingRectForGlyphIndex:glyphIndex];
            [window drawInsertionPointInRect:glyphRect color:aColor];
            [context restoreGraphicsState];
            return;
        }

        ...omit...
 
    if ([self isIDEPlaygroundSourceTextView]) {
        [self _drawInsertionPointInRect:NSZeroRect color:[NSColor grayColor]];
        return;
    }

```

首先是判断是是否处于 Playground 中，如果是，用一个 NSRect 在目前位置画出一个灰色光标。

从上面的判断可以看出 Playground 确实不是从 NSTextView 继承而来，而是一个 IDEPlaygroundTextView 的衍生类，所以画光标的方式也要发生变化。