---
id: 1564
title: 'Learn Python with Steem #07 笔记'
date: '2018-08-07T10:26:12+08:00'
author: hacper
excerpt: "## 划重点\n\n- 类与对象\n\n 把一些具有共同特征的对象的属性和行为（方法）抽取出来，将其**抽象化**，定义为类。\n 也就是说类是对象的模板，按照模板（类）**实例化（具体化）**，这就是对象。\n 类是抽象的概念，而对象是具体的东西。\n \n\n- 类的属性与方法\n\n 类的属性和方法都是一群对象的共同特征。\n 属性是那些对象的静态特征，在类中定义的变量。\n 方法是那些对象的动态特征（行为），在类中定义的函数。\n 这么说也不太准确，有些类中的有些方法和属性是关于类的，而与对象无关系。\n 通过定义类，实现了对数据和对数据的操作的**封装**。"
layout: post
guid: 'https://hacperme.com/?p=1564'
permalink: /2018/08/07/learn-python-with-steem-07-note/
Steempress_sp_steem_publish:
    - '1'
views:
    - '907'
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - da-learnpythonwithsteem
    - python
---

# Learn Python with Steem #07 笔记

- - - - - -

\[toc\]

## 划重点

- 类与对象 把一些具有共同特征的对象的属性和行为（方法）抽取出来，将其**抽象化**，定义为类。  
  也就是说类是对象的模板，按照模板（类）**实例化（具体化）**，这就是对象。  
  类是抽象的概念，而对象是具体的东西。
- 类的属性与方法
  
  类的属性和方法都是一群对象的共同特征。  
  属性是那些对象的静态特征，在类中定义的变量。  
  方法是那些对象的动态特征（行为），在类中定义的函数。  
  这么说也不太准确，有些类中的有些方法和属性是关于类的，而与对象无关系。  
  通过定义类，实现了对数据和对数据的操作的**封装**。

## 编程练习

```python
# 作业
class Person(object):
    def __init__(self, _name, _height, _weight, _gender):
        self.name = _name
        self.height = _height
        self.weight = _weight
        self.gender = _gender
        self.count_update = 0

    def get_bmi(self):
        bmi = sel.weight / (self.height**2)
        return bmi

    def rename(self, _name):
        if self.name == _name:
            pass
        else:
            self.name = _name
            self.count_update += 1


me = Person('zzzABC', 1.75, 65, 'Male')
print(me.name)
me.rename('zzzABC')
me.rename('kkkABC')
print('========= After Update ===========')
print(me.name)
print(me.count_update)

```

```
zzzABC
========= After Update ===========
kkkABC
1

```

## 补充

### 对象属性和类属性

```python
class People(object):
    # 定义类属性count_people，与所有对象共享
    count_people = 0

    # 创建对象时的初始化操作
    def __init__(self, _name):
        # 定义对象属性name
        self.name = _name
        People.count_people += 1

    # 删除对象时的操作
    def __del__(self):
        People.count_people -= 1


me = People('Yjcps')
print(me.name)
print(me.count_people)
print()

friend = People('Tom')
print(friend.name)
print(friend.count_people)
print(me.count_people)
print()

del friend
print(me.count_people)
print()

del me
print(People.count_people)


```

```
Yjcps
1

Tom
2
2

1

0

```

### 静态方法和类方法

```python
class Triangle(object):
    def __init__(self, a, b, c):
        self._a = a
        self._b = b
        self._c = c

    # 静态方法，通过装饰器@staticmethod定义，静态方法属于类方法
    @staticmethod
    def is_valid(a, b, c):
        return a + b > c and b + c > a and a + c > b

    # 类方法使用装器@classmethod定义，cls 表示类本身
    @classmethod
    def create_by_default(cls):
        a = 3
        b = 4
        c = 5
        return cls(a, b, c)

    # 对象的方法，self表示对象本身
    def perimeter(self):
        return self._a + self._b + self._c


# 检查是否可以构成三角形
if Triangle.is_valid(1, 2, 3):
    t1 = Triangle(1, 2, 3)
else:
    print("Can't creat a Triangle")

# 通过类方法创建默认的三角形
t2 = Triangle.create_by_default()

# 通过对象的方法计算三角形的周长
print('perimeter:', t2.perimeter())

```

```
Can't creat a Triangle
perimeter: 12

```

### 抽象方法

```python
import abc


class Shape(object):
    def __init__(self, _name):
        self.name = _name

    # 定义没有（可能无法）实现的方法，需要在子类中改写此方法
    @abc.abstractmethod
    def perimeter(self):
        '''Method should be rewrited'''


t1 = Shape('Shape')
t1.perimeter()

```

### 继承

```python
import abc


# 定义父类（基类）
class Shape(object):
    def __init__(self, _name):
        self.name = _name

    # 定义没有（可能无法）实现的方法，需要在子类中改写此方法
    @abc.abstractmethod
    def perimeter(self):
        '''Method should be rewrited'''


# 定义子类 继承Shape类
class Triangle(Shape):
    def __init__(self, a, b, c):
        # 调用父类的初始化方法
        Shape.__init__(self, 'Triangle')

        self._a = a
        self._b = b
        self._c = c

    # 在子类中改写perimeter方法
    def perimeter(self):
        return self._a + self._b + self._c


t1 = Triangle(3, 4, 5)
print(t1.name)
print(t1.perimeter())

```

```
Triangle
12

```

### 多态

```python
class Square(Shape):
    def __init__(self, a):
        # 调用父类的初始化方法
        Shape.__init__(self, 'Square')
        self._a = a

    # 在子类中改写perimeter方法

    def perimeter(self):
        return self._a**2


t1 = Triangle(3, 4, 5)
s1 = Square(5)

# 不同的类可以有相同名字的方法，它们的行为可以不同，这就是多态。
print(t1.perimeter())
print(s1.perimeter())

```

```
12
25

```

- - - - - -

## \[DA series - Learn Python with Steem\]

- [\[DA series - Learn Python with Steem #01\] 安裝Python、文字編輯器與哈囉！](https://busy.org/@deanliu/da-series-learn-python-with-steem-01-python)
- [\[DA series - Learn Python with Steem #02\] 變數與資料型態](https://busy.org/@deanliu/da-series-learn-python-with-steem-02)
- [\[DA series - Learn Python with Steem #03\] 邏輯判斷](https://busy.org/@deanliu/da-series-learn-python-with-steem-03)
- [\[DA series - Learn Python with Steem #04\] 迴圈](https://busy.org/@deanliu/da-series-learn-python-with-steem-04)
- [\[DA series – Learn Python with Steem #05\] 基本資料結構](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [\[DA series - Learn Python with Steem #06\] 函式](https://busy.org/@deanliu/da-series-learn-python-with-steem-06)
- [\[DA series - Learn Python with Steem #07\] 類別](https://busy.org/@deanliu/da-series-learn-python-with-steem-07)

## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)
- [Learn Python with Steem #03 笔记](https://busy.org/@yjcps/learnpythonwithsteem03-l5w15fszh9)
- [Learn Python with Steem #04 笔记](https://busy.org/@yjcps/learnpythonwithsteem04-0d8jly8ypt)
- [Learn Python with Steem #05 笔记](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [Learn Python with Steem #06 笔记](https://busy.org/@yjcps/learnpythonwithsteem06-c0tgbg3puu)