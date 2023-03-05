---
id: 1382
title: 用mathematica求解模2因式分解
date: '2018-06-14T11:30:42+08:00'
author: hacper
excerpt: "在求循环码的生成多项式和一致性校验多项式时，需要用到模2因式分解，但是自己数学没学好，一个模2因式分解让我摸不着头脑，不知道从何下手。\n\n幸运的是我发现可以用wolfram的mathematica软件来辅助求解，使用Factor函数即可计算模2因式分解。"
layout: post
guid: 'https://hacperme.com/?p=1382'
permalink: /2018/06/14/mathematica_factor_mod_2/
views:
    - '3053'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - ''
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - Factor
    - mathematica
    - 模2因式分解
---

在求循环码的生成多项式和一致性校验多项式时，需要用到模2因式分解，但是自己数学没学好，一个模2因式分解让我摸不着头脑，不知道从何下手。

幸运的是我发现可以用wolfram的mathematica软件来辅助求解，使用Factor函数即可计算模2因式分解。

这是Factor的语法：

![图片.png](https://steemitimages.com/p/7ohP4GDMGPrUMp8dW6yuJTR9MKNu8P8DCXDU9qmmpmrK7Qn1iFHtcnXf2kcotuXK3Z7MyDusxoTYUDrb97v8wARVfYi3D2rVo9eM)

打开mathematica软件试试效果：

![图片.png](https://steemitimages.com/p/7ohP4GDMGPrUMp8dW6yuJTR9MKNu8P8DCXDU9qmmhxwhSHuMa4JE6rjvmZMmywggvjGRQtQ4ATZFjcDuCzqXWm6agKz5VdRgMUuv)

问题解决了？

并非如此，软件只是单纯地告诉我一个结果，我还是不知道如何进行模2因式分解！