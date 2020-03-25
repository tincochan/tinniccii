---
layout: post
title: "number inputs的替换方案"
description: "input type=number的替换方案"
date: 2020-03-25
tags: [前端]
---

```<input type="number">```的作用不需要多讲，但是有一种情况就是比如说银行的app，当你输入自己的信用卡账号的时候，你希望十几位的数字没输入4个数字的时候app输入框显示会自动插入空格。

曾经我在耐克实习的时候就碰到过这样的需求，这里特别针对手机端的前端要求。

<!--more-->

使用```inputmode```属性的替换方案不错，但是版本过低的浏览器和IE并不兼容这一标签：

```html
<input type="text" inputmode="numeric" pattern="[0-9]*">
```

```inputmode```能够保证手机端能够根据输入框的属性弹出适合的数字输入键盘。```type="text" inputmode="numeric"```比起单纯的```type="number"```（比如```<a>``` ```<code>```）来说更能够胜任特定的场景需要，因为前者有诸如```maxlength```， ```minlength```或者```pattern```这样的属性定制输入框的内容。

![inputmode-01](D:/tinniccii/images/2020-3-25/inputmode-01.png)

补充知识：[html针对不同场景相关的自动填充属性列表链接](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete#Values)。