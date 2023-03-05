---
id: 1543
title: 'Learn Python with Steem #04 笔记'
date: '2018-07-31T20:44:30+08:00'
author: hacper
excerpt: "## 划重点\n\n- 循环\n\n 人做重复性的事情很低效和枯燥，相比之下让机器来完成机器更适合，所以在大鹏的《学R》中说，循环是个救世主。\n 循环由循环条件和循环体组成：满足条件，程序就重复做一类事情，要重复做的事情就是循环体，用缩进体现。\n\n- 循环的类型 \n\n 循环可分两类：已知重复次数，用 for 循环方便一些，用while循环也可以，for循环还可以对一个容器进行迭代。 \n 不知道具体循环次数或者需要一个死循环，可以使用while循环,直到发现不满足循环条件，就退出循环。\n\n- 字符串格式化\n\n 格式化字符串用format()函数，也可以用占位符%。"
layout: post
guid: 'https://hacperme.com/?p=1543'
permalink: /2018/07/31/learn-python-with-steem-04-note/
Steempress_sp_steem_publish:
    - '1'
views:
    - '975'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - learnpythonwithsteem04-0d8jly8ypt
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - da-learnpythonwithsteem
    - python
---

# Learn Python with Steem #04 笔记

- - - - - -

\[toc\]

## 划重点

- 循环

人做重复性的事情很低效和枯燥，相比之下让机器来完成这些事更适合，所以在大鹏的《学R》中说，循环是个救世主。

循环由循环条件和循环体组成：满足条件，程序就重复做一类事情，要重复做的事情就是循环体，用缩进体现。

- 循环的类型 循环可分两类：已知重复次数，用 for 循环方便一些，用while循环也可以，for循环还可以对一个容器进行迭代。  
  不知道具体循环次数或者需要一个死循环，可以使用while循环,直到发现不满足循环条件，就退出循环。
- 字符串格式化
  
  格式化字符串用format()函数，也可以用占位符%。

## 编程练习

```python
# 作业 

for row in range(1, 10):
    for col in range(1, row + 1):
        print('{}*{}={}'.format(row, col, row * col), end='\t')
    print()

```

```
1*1=1   
2*1=2   2*2=4   
3*1=3   3*2=6   3*3=9   
4*1=4   4*2=8   4*3=12  4*4=16  
5*1=5   5*2=10  5*3=15  5*4=20  5*5=25  
6*1=6   6*2=12  6*3=18  6*4=24  6*5=30  6*6=36  
7*1=7   7*2=14  7*3=21  7*4=28  7*5=35  7*6=42  7*7=49  
8*1=8   8*2=16  8*3=24  8*4=32  8*5=40  8*6=48  8*7=56  8*8=64  
9*1=9   9*2=18  9*3=27  9*4=36  9*5=45  9*6=54  9*7=63  9*8=72  9*9=81  

```

```python
for row in range(1, 10):
    for col in range(1, row + 1):
        print('%d*%d=%d' % (row, col, row * col), end='\t')
    print()


```

```
1*1=1   
2*1=2   2*2=4   
3*1=3   3*2=6   3*3=9   
4*1=4   4*2=8   4*3=12  4*4=16  
5*1=5   5*2=10  5*3=15  5*4=20  5*5=25  
6*1=6   6*2=12  6*3=18  6*4=24  6*5=30  6*6=36  
7*1=7   7*2=14  7*3=21  7*4=28  7*5=35  7*6=42  7*7=49  
8*1=8   8*2=16  8*3=24  8*4=32  8*5=40  8*6=48  8*7=56  8*8=64  
9*1=9   9*2=18  9*3=27  9*4=36  9*5=45  9*6=54  9*7=63  9*8=72  9*9=81  

```

```python
# 猜数游戏
import random

answer = random.randint(1, 100)
counter = 7
while True:
    if counter > 0:
        print('你还剩%d次机会' % counter)
        number = int(input('请输入答案: '))
        counter -= 1
        if number < answer:
            print('太小了')
        elif number > answer:
            print('太大了')
        else:
            print('恭喜你猜对了!')
            break
    else:
        print('Game Over!')
        break


```

```
你还剩7次机会
请输入答案: 50
太小了
你还剩6次机会
请输入答案: 80
太大了
你还剩5次机会
请输入答案: 70
太小了
你还剩4次机会
请输入答案: 75
太小了
你还剩3次机会
请输入答案: 78
太大了
你还剩2次机会
请输入答案: 76
太小了
你还剩1次机会
请输入答案: 77
恭喜你猜对了!

```

## 补充

### 循环控制

```python
# break 跳出循环，终止循环
for i in range(5):
    if i == 3:
        break
    print(i)

```

```
0
1
2

```

```python
# continue 放弃本次循环，但不终止循环
for i in range(5):
    if i == 3:
        continue
    print(i)

```

```
0
1
2
4

```

## 输出重定向

打印99乘法表的时候查看了print()函数的参数：

> Docstring:  
>  print(value, ..., sep=' ', end='\\n', file=sys.stdout, flush=False)
> 
>  Prints the values to a stream, or to sys.stdout by default.

file=sys.stdout 这个参数没有用过，它默认把输出打印到屏幕（sys.stdout），应该也可以把输出重定向到其他地方，比如文件，  
试试看

```python
f = open('K:\Workspace\python\Python-x-Steem-tutorial-master\Python-x-Steem-tutorial\hello.txt', 'w')
print('Hello balabala!!', file=f)
f.close()


```

```python
%pycat hello.txt


```

结果：  
![blob.jpg](https://i.loli.net/2018/07/31/5b6054cfaca3e.jpg)  
![blob.jpg](https://i.loli.net/2018/07/31/5b6055162315c.jpg)

- - - - - -

\## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)
- [Learn Python with Steem #03 笔记](https://busy.org/@yjcps/learnpythonwithsteem03-l5w15fszh9)