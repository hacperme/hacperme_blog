---
id: 1557
title: 'Learn Python with Steem #06 笔记'
date: '2018-08-04T00:10:28+08:00'
author: hacper
excerpt: "## 划重点\n\n- 函数\n\n 函数是实现某个特定功能的一段代码。\n \n 将同一类相对独立的功能写成一个个函数，放到同一个py文件里，这就是一个模块。\n \n 你可以试试输入 help(math) 看是不是这样的。\n \n\n- 函数的结构\n\n 函数由输入参数、函数体、输出参数组成。\n \n 通过输入参数接收要处理的数据，\n \n 在函数体里实现要完成的功能，\n \n 最后将计算结果通过输出参数返回给使用者。"
layout: post
guid: 'https://hacperme.com/?p=1557'
permalink: /2018/08/04/learn-python-with-steem-06-note/
Steempress_sp_steem_publish:
    - '1'
views:
    - '1030'
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - da-learnpythonwithsteem
    - python
---

# Learn Python with Steem #06 笔记

- - - - - -

\[toc\]

## 划重点

- 函数 函数是实现某个特定功能的一段代码。
  
  将同一类相对独立的功能写成一个个函数，放到同一个py文件里，这就是一个模块。
  
  你可以试试输入 help(math) 看是不是这样的。
- 函数的结构
  
  函数由输入参数、函数体、输出参数组成。
  
  通过输入参数接收要处理的数据，
  
  在函数体里实现要完成的功能，
  
  最后将计算结果通过输出参数返回给使用者。

## 编程练习

```python
# 作业
def my_average(data):
    average = (sum(data) - max(data) - min(data)) / (len(data) - 2)
    return average


my_list = [1, 8, 3, 6, 2, 345, -23, 7]
answer = my_average(my_list)
print(answer)

```

```
4.5

```

```python
# 将计算busy机器人点赞比例的功能写成一个函数
import requests


def busy_vote_percent(username):

    minVests = 20000000
    # maxVests = 4000000000000  # maxVests 不是这个值
    maxVests = 5000000000000
    limitVests = 10000000000000
    minPercent = 6 / 100
    maxPercent = 2500 / 100

    url = 'https://steemdb.com/api/accounts?account=' + username
    response = requests.get(url)
    data = response.json()[0]
    followers_mvest = data.get('followers_mvest')
    if followers_mvest < minVests or followers_mvest > limitVests:
        percent = 0
    else:
        percent = (10000 / maxVests) * followers_mvest
        #         print(percent)
        percent = min(max(percent, minPercent), maxPercent) / 100

    print('Hi,{}\nfollowers_mvest:{}\nyour busy vote percent is: {:.2%}\n'.
          format(username, followers_mvest, percent))


#     return percent

busy_vote_percent('yjcps')
busy_vote_percent('dapeng')
busy_vote_percent('shine.wong')
busy_vote_percent('deanliu')

```

```
Hi,yjcps
followers_mvest:93266616.60498305
your busy vote percent is: 0.19%

Hi,dapeng
followers_mvest:1856251363.8415272
your busy vote percent is: 3.71%

Hi,shine.wong
followers_mvest:261131007.75513688
your busy vote percent is: 0.52%

Hi,deanliu
followers_mvest:18379826592.987816
your busy vote percent is: 25.00%

```

## 补充

### 变量的作用域

变量的作用域指的是变量的可见范围，分两类：

- 全局变量
- 局部变量

```python
# 全局变量
a = 20
b = 30
c = 5


def my_fun():
    # 局部变量
    a = 7
    b = 10
    print('call my_fun：a={}, b={}, c={}'.format(a, b, c))

#     在函数里面有定义a,b变量属于局部变量，
#     在使用的时候优先使用局部变量，
#     看不见函数外面的全局变量

#     函数里面没有定义c变量
#     使用的时候会去查找外面的全局变量


my_fun()

print('a={}, b={}, c={}'.format(a, b, c))

# 在外面看不见函数里面的局部变量

```

```
call my_fun：a=7, b=10, c=5
a=20, b=30, c=5

```

```python
# 全局变量
a = 20
b = 30
c = 5


def my_fun():

    # 局部变量
    a = 7

    # 可以用global声明b是函数外面的全局变量
    global b
    b = 10
    print('call my_fun：a={}, b={}, c={}'.format(a, b, c))


my_fun()

print('a={}, b={}, c={}'.format(a, b, c))

# 在函数外面可以看到变量b被修改了

```

```
call my_fun：a=7, b=10, c=5
a=20, b=10, c=5

```

### 匿名函数

Python中用lambda定义匿名函数

格式： lambda 参数:表达式

```python
f = lambda x: x * 2 + x + 2

## 等价f(x)
# def f(x):
#     return x * 2 + x + 2

print(f(2))
print(f(5))

```

```
8
17

```

### 递归函数

在函数里面调用函数自己本身来解决问题，这就是递归。

在遇到一些定义，数据结构或问题的解决方法是递归的时候，可以考虑使用递归函数。

```python
# 经典例子：计算n的阶乘n!


def factorial(n):

    # 递归出口
    if n == 0:
        return 1
    # 递归体
    return n * factorial(n - 1)


print(factorial(4))
print(factorial(9))

```

```
24
362880

```

### 装饰器

在Python中可以用装饰器来修饰函数。

使用装饰器的目的在于：为原有函数添加新的功能但不修改原来的程序。

```python
# 定义一个装饰器
def print_fun_name(func):
    def wrapper(*args, **kw):
        print('-----------------')
        print('call function:{}'.format(func.__name__))
        print('-----------------')
        return func(*args, **kw)

    return wrapper


# 使用装饰器


@print_fun_name
def say_hello(name):
    print('Hello', name)


@print_fun_name
def say_bye(name):
    print('Bye', name)


say_hello('yjcps')
say_bye('hacper')

```

```
-----------------
call function:say_hello
-----------------
Hello yjcps
-----------------
call function:say_bye
-----------------
Bye hacper

```

- - - - - -

## \[DA series - Learn Python with Steem\]

- [\[DA series - Learn Python with Steem #01\] 安裝Python、文字編輯器與哈囉！](https://busy.org/@deanliu/da-series-learn-python-with-steem-01-python)
- [\[DA series - Learn Python with Steem #02\] 變數與資料型態](https://busy.org/@deanliu/da-series-learn-python-with-steem-02)
- [\[DA series - Learn Python with Steem #03\] 邏輯判斷](https://busy.org/@deanliu/da-series-learn-python-with-steem-03)
- [\[DA series - Learn Python with Steem #04\] 迴圈](https://busy.org/@deanliu/da-series-learn-python-with-steem-04)
- [\[DA series – Learn Python with Steem #05\] 基本資料結構](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [\[DA series - Learn Python with Steem #06\] 函式](https://busy.org/@deanliu/da-series-learn-python-with-steem-06)

## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)
- [Learn Python with Steem #03 笔记](https://busy.org/@yjcps/learnpythonwithsteem03-l5w15fszh9)
- [Learn Python with Steem #04 笔记](https://busy.org/@yjcps/learnpythonwithsteem04-0d8jly8ypt)
- [Learn Python with Steem #05 笔记](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)