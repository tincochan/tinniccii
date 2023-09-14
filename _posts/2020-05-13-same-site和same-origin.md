---
layout: post
title: "same-site和same-origin"
description: "弄明白这两个的区别"
date: 2020-05-13
tags: [Fullstack]
---

必须要说same-site和same-origin这两个概念是有区别的，我以前经常弄混...

<!--more-->

#### Origin（源）

<img src="../images/2020-05-13/same-origin.png" alt="same-origin" style="zoom:50%;" />

Origin是由协议（scheme/protocol，比如http或者https），主机名（host name）以及端口号（port，不一定有）构成，比如一个url为```https://www.example.com:443/foo```，那么这个url的origin就是```https://www.example.com:443```。

因此，不同的网站如果协议、主机名以及端口号都是相同的，那么它们之间就是"same-origin"(同源)，否则称之为"cross-origin"(跨域)。

| Origin A                    | Origin B                          | 关系                                     |
| --------------------------- | --------------------------------- | ---------------------------------------- |
| https://www.example.com:443 | https://**www.evil.com**:443      | 跨域：域名不同（different domains）      |
|                             | https://**example.com**:443       | 跨域：子域名不同（different subdomains） |
|                             | https://**login**.example.com:443 | 跨域：子域名不同                         |
|                             | **http**://www.example.com:443    | 跨域：协议不同                           |
|                             | https://www.example.com:**80**    | 跨域：端口号不同                         |
|                             | **https://www.example.com:443**   | 同源                                     |
|                             | **https://www.example.com**       | 同源（默认端口号一致）                   |

#### Site（站）

<img src="../images/2020-05-13/same-site.png" alt="same-site" style="zoom:50%;" />

在[Root Zone Database](https://www.iana.org/domains/root/db)数据库里面列出了所有比如```.com```和```.org```这样的顶级域（Top-level domains , TLDs）。Site的意思就是TLDs和它前面的域（domain）的一部分所构成。比如对于url```https://www.example.com:443/foo```来说，它的site就是```example.com```。

但是比如说```.co.jp```或者是```.github.io```这样的域来说，如果是按照上面的定义从```.jp```或者是```.io```这样的TLDs出发去定义，显然这样并不准确。这样就需要一个标准[Public Suffix List](https://wiki.mozilla.org/Public_Suffix_List)去定义网址中有效的TLDs（effective TLDs, eTLDs），这个列表[publicsuffix.org/list](https://publicsuffix.org/list/)列出了所有的eTLDs。

因此整个网址名也就是site可以看作eTLD+1，比如url```https://my-project.github.io```，那么它的eTLD就是```.github.io```，eTLD+1也就是site即```my-project.github.io```。

<img src="../images/2020-05-13/same-site.png" alt="same-site" style="zoom:50%;" />

所以网站之间如果有相同的eTLD+1，那么它们就是same-site（同站），反之就是cross-site（跨站）。

| Origin A                    | Origin B                          | 关系                                |
| --------------------------- | --------------------------------- | ----------------------------------- |
| https://www.example.com:443 | https://**www.evil.com**:443      | 跨站：域名不同（different domains） |
|                             | https://**example.com**:443       | 以下皆为同站                        |
|                             | https://**login**.example.com:443 |                                     |
|                             | **http**://www.example.com:443    |                                     |
|                             | https://www.example.com:**80**    |                                     |
|                             | **https://www.example.com:443**   |                                     |
|                             | **https://www.example.com**       |                                     |

<img src="../images/2020-05-13/schemeful-same-site.png" alt="schemeful-same-site" style="zoom:50%;" />

以上所说的same-site是没有考虑协议（scheme/protocol）的，但是因为安全性问题，same-site有时需要考虑协议相同，也就是schemeful same-site，因此```http://www.example.com```和```https://www.example.com```视为cross-site。

#### 如何判断请求request是"same-site", "same-origin", 还是 "cross-site"？

Chrome在发送请求时会带有```Sec-Fetch-Site```这样的HTTP header，目前其他浏览器并不支持```Sec-Fetch-Site```。属性值可以对请求进行相应的判断。



