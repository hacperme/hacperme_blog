---
id: 1539
title: 'Learn Python with Steem #03 笔记'
date: '2018-07-31T20:29:25+08:00'
author: hacper
excerpt: "## 划重点\n\n- 分支结构\n\n Python的分支结构用if、elif、else关键字来构造，可以是多分支，也可以嵌套。\n\n- 代码块\n\n Python中用缩进的方式构造代码块，程序的层次结构一目了然。\n\n- 交互\n\n 使用input()函数输入数据，实现人与程序的交互。"
layout: post
guid: 'https://hacperme.com/?p=1539'
permalink: /2018/07/31/learn-python-with-steem-03-note/
Steempress_sp_steem_publish:
    - '1'
views:
    - '949'
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - da-learnpythonwithsteem
    - python
---

# Learn Python with Steem #03 笔记

- - - - - -

\[toc\]

## 划重点

- 分支结构 Python的分支结构用if、elif、else关键字来构造，可以是多分支，也可以嵌套。
- 代码块
  
  Python中用缩进的方式构造代码块，程序的层次结构一目了然。
- 交互
  
  使用input()函数输入数据，实现人与程序的交互。

## 编程练习

```python
# 写作业

a = float(input('Please enter the first number: '))
b = float(input('Please enter the second number: '))
c = float(input('Please enter the last number: '))

if a > b and a > c:
    print(a)
elif b > a and b > c:
    print(b)
else:
    print(c)

```

```
Please enter the first number: 55
Please enter the second number: 99
Please enter the last number: 22
99.0

```

计算busy机器人的点赞比例，这是它的js程序[https://github.com/busyorg/busy-bot/](https://github.com/busyorg/busy-bot/blob/master/src/upvoter/index.js)

```js
function getAccounts() {
  const accounts = JSON.parse(process.env.STEEM_ACCOUNTS || '[]');

  return accounts.map(account => ({
    username: account.username,
    wif: account.wif,
    minVests: account.minVests || 20000000,
    maxVests: account.maxVests || 4000000000000,
    limitVests: account.limitVests || 10000000000000,
    minPercent: account.minPercent || 6,
    maxPercent: account.maxPercent || 2500,
  }));
}

async function getVoteWeight(username, account) {
  const mvests = await fetch(`https://steemdb.com/api/accounts?account[]=${username}`)
    .then(res => res.json())
    .then(res => res[0].followers_mvest);

  if (mvests < account.minVests || mvests > account.limitVests) return 0;

  const percent = parseInt((10000 / account.maxVests) * mvests);

  return Math.min(Math.max(percent, account.minPercent), account.maxPercent);
}



```

```python
<br></br># 这算是一个实际的例子吧，把它的计算部分抽出来，
# 用Python写一下，正好用到今天学的分支结构
# 查看followers_mvest
# https://steemdb.com/api/accounts?account=yjcps

minVests = 20000000
# maxVests = 4000000000000  # maxVests 不是这个值
maxVests = 5000000000000
limitVests = 10000000000000
minPercent = 6 / 100
maxPercent = 2500 / 100

followers_mvest = float(input('Input your followers_mvest: '))
# 93785952.40679602

if followers_mvest < minVests or followers_mvest > limitVests:
    percent = 0
else:
    percent = (10000 / maxVests) * followers_mvest
    percent = min(max(percent, minPercent), maxPercent) / 100
print('Your Busy vote percent: {:.2%}'.format(percent))


```

```
Input your followers_mvest: 93785952.40679602
Your Busy vote percent: 0.19%

```

## 补充

### 比较运算符

<= < > >= 小于等于，小于，大于，大于等于

== != 等于，不等于

### 逻辑运算符

not or and 非，或，与

### 空语句 pass

pass 不是跳过某段程序的意思,它是是用来占位子的，为了让写的程序符合语法。

pass有两个用法用：  
一是什么都不做，写个pass：

```python
<br></br>if True:
   pass


```

另一个是，还没想好程序的某个功能怎么写，写个pass占位子:

```python
<br></br>def fun():
    pass


```

- - - - - -

## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)