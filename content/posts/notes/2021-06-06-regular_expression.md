---
id: 1951
title: 正则表达式备忘清单
date: '2021-06-06T22:43:57+08:00'
author: hacper
layout: post
guid: 'https://hacperme.com/?p=1951'
permalink: /2021/06/06/regular_expression/
views:
    - '1117'
categories:
    - 笔记
tags:
    - 正则表达式
---

## 定位

![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting/raw/master/20210507/定位.yjf3jw4m6eo.png)

## 表达式

![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting/raw/master/20210507/表达式.1jareij8pwlc.png)

## 量词

![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting/raw/master/20210507/量词1.4rsfwbeuejs0.png)

![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting/raw/master/20210507/量词2.3w50kxb49dk0.png)

![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting/raw/master/20210507/量词3.69qjn4ikif80.png)

## 字符组

![](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting/raw/master/20210507/字符组.6aekjt1hkic0.png)

| <th style="text-align: left;">POSIX 字符集</th> | <th style="text-align: left;">描述</th> |
|----------------------------------------------------|-------------------------------------------|
| \[\[:alnum:\]\] | 字母数字字符 （字母和数字） |
| \[\[:alpha:\]\] | 字母字符（字母） |
| \[\[:ascii:\]\] | ASCII字符 （总共128个） |
| \[\[:blank:\]\] | 空白字符 |
| \[\[:ctrl:\]\] | 控制字符 |
| \[\[:digit:\]\] | 数字 |
| \[\[:graph:\]\] | 图形字符 |
| \[\[:lower:\]\] | 小写字母 |
| \[\[:print:\]\] | 可打印字符 |
| \[\[:punct:\]\] | 标点符号 |
| \[\[:space:\]\] | 空格字符 |
| \[\[:upper:\]\] | 大写字母 |
| \[\[:word:\]\] | 单词字符 |
| \[\[:xdigit:\]\] | 十六进制数字 |

## 关于量词的贪心、懒惰、占有

- 量词首次尝试匹配整个字符串，如果失败则回退一个字符后再次尝试，这个过程叫做回溯（backtracking）。
- 量词自身是贪心的。
- 贪心的量词会首先匹配整个字符串。尝试匹配时，它会选定尽可能多的内容，也就是整个输入，它会每次回退一个字符，直到找到匹配的内容或者没有字符可尝试为止。此外，它还记录所有的行为，因此资源的消耗较大。
- 懒惰的量词会从目标的起始位置开始尝试寻找匹配，每次检查字符串的一个字符，寻找它要匹配的内容，最后，它会尝试匹配整个字符串。
- 占有量词会覆盖整个目标然后尝试寻找匹配内容，但它只尝试一次，不会回溯。