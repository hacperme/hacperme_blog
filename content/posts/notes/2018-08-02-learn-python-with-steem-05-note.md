---
id: 1549
title: 'Learn Python with Steem #05 笔记'
date: '2018-08-02T12:40:11+08:00'
author: hacper
excerpt: "## 划重点\n\n- 字典\n\n 字典是Python内置的一种可变的数据结构，也是一种容器，可以放置任意类型的元素，\n \n 特点是用 {} 定义，元素以键值对的形式组织，键和值是一一对应的映射关系。\n \n 字典的常用操作：创建，访问，更新，删除。\n \n\n- 列表\n\n 列表也是Python中一种可变的数据结构，其元素可以是任意类型，用 [] 定义。\n \n 列表的常用操作：创建，访问，添加，删除，切片，排序。"
layout: post
guid: 'https://hacperme.com/?p=1549'
permalink: /2018/08/02/learn-python-with-steem-05-note/
Steempress_sp_steem_publish:
    - '1'
views:
    - '1065'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - learnpythonwithsteem05-n09ajzsh91
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - python
    - steem
---

# Learn Python with Steem #05 笔记

- - - - - -

\[toc\]

## 划重点

- 字典 字典是Python内置的一种可变的数据结构，也是一种容器，可以放置任意类型的元素，
  
  特点是用 {} 定义，元素以键值对的形式组织，键和值是一一对应的映射关系。
  
  字典的常用操作：创建，访问，更新，删除。
- 列表
  
  列表也是Python中一种可变的数据结构，其元素可以是任意类型，用 \[\] 定义。
  
  列表的常用操作：创建，访问，添加，删除，切片，排序。

## 编程练习

```python
# 作业
my_list = [n**3 for n in range(1, 11)]
print(my_list)

my_dictionary = dict(zip(range(1, 11), [n**2 for n in range(1, 11)]))
print(my_dictionary)

```

```
[1, 8, 27, 64, 125, 216, 343, 512, 729, 1000]
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36, 7: 49, 8: 64, 9: 81, 10: 100}

```

## 补充

### 列表

```python
# 创建

# 元素可以是任意类型
my_list = [3, 'apple', 3.14, [1, 2, 3], {'name': 'yjcps'}]
print(my_list)

# 列表生成式
my_num = [n+1 for n in range(10)]
print(my_num)

```

```
[3, 'apple', 3.14, [1, 2, 3], {'name': 'yjcps'}]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

```

```python
# 使用整数下标访问元素，第一个元素从0开始
print(my_list[0])
print(my_list[2])
# 可以使用负数
print(my_list[-3])

```

```
3
3.14
3.14

```

```python
# 元素个数，列表长度
print(len(my_list))

```

```
5

```

```python
# 添加元素
my_list.append(555) # 添加到末尾
print(my_list)
my_list.insert(-2, 'steem') #指定位置
print(my_list)
my_list.extend([16,22,73,45])
print(my_list)

```

```
[3, 'apple', 3.14, [1, 2, 3], {'name': 'yjcps'}, 555]
[3, 'apple', 3.14, [1, 2, 3], 'steem', {'name': 'yjcps'}, 555]
[3, 'apple', 3.14, [1, 2, 3], 'steem', {'name': 'yjcps'}, 555, 16, 22, 73, 45]

```

```python
# 遍历列表
for item in my_list:
    print(item)

```

```
3
apple
3.14
[1, 2, 3]
steem
{'name': 'yjcps'}
555
16
22
73
45

```

```python
# 删除元素
my_list.remove(3.14)
print(my_list)
del my_list[-1]
print(my_list)
my_list.remove('apple')
print(my_list)

print()

my_list.clear() # 清空
print(my_list)

```

```
[3, 'apple', [1, 2, 3], 'steem', {'name': 'yjcps'}, 555, 16, 22, 73, 45]
[3, 'apple', [1, 2, 3], 'steem', {'name': 'yjcps'}, 555, 16, 22, 73]
[3, [1, 2, 3], 'steem', {'name': 'yjcps'}, 555, 16, 22, 73]

[]

```

```python
# 切片 list[起:s止:步伐]

print(my_num)
print(my_num[2:5])
print(my_num[1:8:2])
print(my_num[3:-3])
print(my_num[-2:3:-1])
print(my_num[::-1])

```

```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
[3, 4, 5]
[2, 4, 6, 8]
[4, 5, 6, 7]
[9, 8, 7, 6, 5]
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

```

```python
# 排序
my_num = [11, 4, 56, 98, 34, 6, 77, 23, 12.4, 3]
print(sorted(my_num))
print(sorted(my_num, reverse=True))
print(my_num) # 不改变原来的列表

print()

my_num.sort(reverse=True)
print(my_num) # 改变原来的列表

```

```
[3, 4, 6, 11, 12.4, 23, 34, 56, 77, 98]
[98, 77, 56, 34, 23, 12.4, 11, 6, 4, 3]
[11, 4, 56, 98, 34, 6, 77, 23, 12.4, 3]

[98, 77, 56, 34, 23, 12.4, 11, 6, 4, 3]

```

### 字典

```python
# 创建

my_dictionary = {
    'A': 1,
    'B': 2,
    'C': 3,
    'D': 4,
    'E': 5
}

print(my_dictionary)

list_keys = ['A', 'B', 'C', 'D', 'E', 'F', 'G']
list_values = [1, 2, 3, 4, 5, 6, ]

my_dictionary = dict(zip(list_keys, list_values))
print(my_dictionary)

```

```
{'A': 1, 'B': 2, 'C': 3, 'D': 4, 'E': 5}
{'A': 1, 'B': 2, 'C': 3, 'D': 4, 'E': 5, 'F': 6}

```

```python
# 访问
print(my_dictionary['B'])

print(my_dictionary.keys())
print(my_dictionary.values())

```

```
2
dict_keys(['A', 'B', 'C', 'D', 'E', 'F'])
dict_values([1, 2, 3, 4, 5, 6])

```

```python
for k, v in my_dictionary.items():
    print(k, v)

print()

for k in my_dictionary:
    print(my_dictionary[k])

```

```
A 1
B 2
C 3
D 4
E 5
F 6

1
2
3
4
5
6

```

```python
# 更新
my_dictionary['A'] = 'one'
print(my_dictionary)

my_dictionary.update({'B': 'two', 'C': 'three'})
my_dictionary.update(F='six')
print(my_dictionary)

```

```
{'A': 'one', 'B': 2, 'C': 3, 'D': 4, 'E': 5, 'F': 6}
{'A': 'one', 'B': 'two', 'C': 'three', 'D': 4, 'E': 5, 'F': 'six'}

```

```python
# 删除
print(my_dictionary.popitem())
print(my_dictionary.popitem())
print(my_dictionary)

print()

print(my_dictionary.pop('B'))
print(my_dictionary)

print()

my_dictionary.clear() # 清空
print(my_dictionary)

```

```
('F', 'six')
('E', 5)
{'A': 'one', 'B': 'two', 'C': 'three', 'D': 4}

two
{'A': 'one', 'C': 'three', 'D': 4}

{}

```

### 集合

集合里的元素是唯一的，无序的，可以行交集、并集、差集等运算

```python
# 创建

my_set_1 = {1,3,4,4,5,6,7,8}
my_set_2 = set(range(10))
print(my_set_1)
print(my_set_2)

```

```
{1, 3, 4, 5, 6, 7, 8}
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

```

```python
# 增删改 

my_set_1.add(99)
my_set_1.add(45)
print(my_set_1)


print(my_set_1.pop())
print(my_set_1.pop())
print(my_set_1)


```

```
{1, 3, 4, 5, 6, 7, 8, 99, 45}
1
3
{4, 5, 6, 7, 8, 99, 45}

```

```python
my_set_1.discard(6)
print(my_set_1)

```

```
{4, 5, 7, 8, 99, 45}

```

```python
my_set_1.remove(8)
print(my_set_1)

```

```
{4, 5, 7, 99, 45}

```

```python
my_set_1.update([8, 15])
print(my_set_1)

```

```
{4, 5, 7, 8, 99, 45, 15}

```

```python
# 遍历

for s in  my_set_1:
    print(s)

```

```
4
5
7
8
99
45
15

```

```python
# 运算

print(my_set_1 & my_set_2)  # 交集
print(my_set_1 | my_set_2)  # 并集
print(my_set_1 - my_set_2)  # 差集
print(my_set_1 ^ my_set_2)  # 对称差集

```

```
{8, 4, 5, 7}
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 15, 99, 45}
{99, 45, 15}
{0, 1, 2, 3, 99, 6, 9, 45, 15}

```

### 元组

元组类似列表，但元组的元素是不可修改的

```python
# 创建
my_tuple = ('yjcps', 123, 'hacper', 55)
print(my_tuple)

```

```
('yjcps', 123, 'hacper', 55)

```

```python
# 访问

print(my_tuple[2])

```

```
hacper

```

```python
# 不可修改
my_tuple[1] = 456

```

```
---------------------------------------------------------------------------

TypeError                                 Traceback (most recent call last)

<ipython-input-24-703d73c28b82> in <module>()
      1 # 不可修改
----> 2 my_tuple[1] = 456


TypeError: 'tuple' object does not support item assignment

```

### 字符串

```python
# 创建字符串
text_1 = 'betty bought a bit of butter but the butter was bitter'
text_2 = "cat bat mat cat bat cat"
text_3 = '''
    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
'''

# 字符串长度
print(len(text_1), len(text_2), len(text_3))

```

```
54 23 214

```

```python
# 首字母大写
print(text_1.capitalize())

# 全部转换为大写
print(text_2.upper())

# 全部转换为小写
print(text_2.lower())

```

```
Betty bought a bit of butter but the butter was bitter
CAT BAT MAT CAT BAT CAT
cat bat mat cat bat cat

```

```python
# 检查字符串的开头结尾
print(text_2.startswith('cat'))
print(text_2.endswith('bat'))

```

```
True
False

```

```python
# 切片，拼接

print(text_3[4:35])
print(text_3[-130:-108])
print(text_3[35:4:-1])

```

```
 Beautiful is better than ugly.
 is better than comple

.ylgu naht retteb si lufituaeB

```

```python
print(text_1 + text_2)
print('#'.join(text_2))

```

```
betty bought a bit of butter but the butter was bittercat bat mat cat bat cat
c#a#t# #b#a#t# #m#a#t# #c#a#t# #b#a#t# #c#a#t

```

```python
# 分割

print(text_2.split())
print(','.join(text_2.split())) 

print()

print(text_3.split('.'))

```

```
['cat', 'bat', 'mat', 'cat', 'bat', 'cat']
cat,bat,mat,cat,bat,cat

['\n    Beautiful is better than ugly', '\n    Explicit is better than implicit', '\n    Simple is better than complex', '\n    Complex is better than complicated', '\n    Flat is better than nested', '\n    Sparse is better than dense', '\n']

```

```python
# 替换

print(text_1.replace('butter', 'hacper'))

```

```
betty bought a bit of hacper but the hacper was bitter

```

```python
# 去除首尾某些字符
text_4 = '   123hello321   '
print(text_4.strip()) # 空格

text_4 = '****123hello321****'
print(text_4.strip('*')) # '*'

```

```
123hello321
123hello321

```

- - - - - -

## \[DA series - Learn Python with Steem\]

- [\[DA series - Learn Python with Steem #01\] 安裝Python、文字編輯器與哈囉！](https://busy.org/@deanliu/da-series-learn-python-with-steem-01-python)
- [\[DA series - Learn Python with Steem #02\] 變數與資料型態](https://busy.org/@deanliu/da-series-learn-python-with-steem-02)
- [\[DA series - Learn Python with Steem #03\] 邏輯判斷](https://busy.org/@deanliu/da-series-learn-python-with-steem-03)
- [\[DA series - Learn Python with Steem #04\] 迴圈](https://busy.org/@deanliu/da-series-learn-python-with-steem-04)
- [\[DA series - Learn Python with Steem #05\] 基本資料結構](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)

## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)
- [Learn Python with Steem #03 笔记](https://busy.org/@yjcps/learnpythonwithsteem03-l5w15fszh9)
- [Learn Python with Steem #04 笔记](https://busy.org/@yjcps/learnpythonwithsteem04-0d8jly8ypt)
- [Learn Python with Steem #05 笔记](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)