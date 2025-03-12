---
title: 从零开始学Python（二）- 字符串和基础操作
date: 2025-03-12 09:58:30
categories:
  - python
tags:
  - python
  - python basics  
---

字符串&操作

<!-- more -->

Python可以操作str类型的字符串，从单个字符"。"到句子"I am python"。

一般字符串都是可以使用成对的单引号或双引号标示

# 单双引号标示


```python
str1 = "123" # 使用了单引号标示，这个时候是字符串（文本）， 而不是数字123
str2 = "I am python"
```

但如果我们的字符串里面本身就需要展示出单引号或双引号呢？

这个时候我们就可以采用双引号包含单引号的方法，或反过来

```python
str1 = "I 'm python" # 双引包含单引
str2 = 'he said,"python is good"' #单引包含双引
```

# 转义

要标示引号本身，还可以采用转义的方法，即加上"\"

```python
>>> "he said,\"python is good\""
"he said,"python is good""
>>>
```

如果我们不使用转义，python会因为无法识别这个语法，给出了SyntaxError的情况

```python
>>> "he said,"python is good""
  File "<stdin>", line 1
    "he said,"python is good""
              ^
SyntaxError: invalid syntax
```

## 应用场景

像我们在自动化测试中常见的JSON字符串的问题，假定我们这些参数放在excel/yaml，在自动化接口测试要提取参数，插件直接给我们返回的字符串没问题，要进行参数化的时候就哦豁了~

```python
{
  "name": "ace020",
}
```
插件读取后，当我们要进行json化
```python
# python: 丸辣，冲我来的，这是个什么造型，双引号包含双引号？算了，给个SyntaxError语法错误吧
"{"name":"ace020"}"  
```

正确做法,假定我们在其他文件中是这样写的

```
{
  \"name\": \"ace020\"
}
```
转为字符串就像str1的值那样
```python
>>> str1 = "{\"name\": \"ace020\"}"
>>> print(str1)
{"name": "ace020"}
# 我们打印str1的类型，显示是str类型的
>>> print(type(str1))
<class "str">
# 这时候可以用json将字符串转换成字典啦
>>> import json
>>> data = json.loads(str1)
>>> print(data)
{'name': 'ace020'}
# 再打印一下类型，成功转为字典啦
>>> print(type(data))
<class 'dict'>
```

# 字符串拼接

## 使用 + 号拼接字符串

```python
>>> str1 = "I am "
>>> str2 = "python"
>>> print(str1 + str2)
I am python
>>> print("Wow " + "amazing")
Wow amazing
```

## 相邻的字符串字面值会自动合并

这里没有使用+号，而是两段字符串，相邻在一起，字符串前后和中段位置，两段字符串使用了4个单引号

```python
>>> text = 'The more efforts you make, ''the more fortune you get.'
>>> print(text)
The more efforts you make, the more fortune you get.
>>>
```

但是要注意，只能用于字面值，不能用于变量，拼接与变量相关时，可以使用加号

```python
# python:又一个语法错误，你们有没有好好学习文档啊
>>> prefix = "py"
# 不使用加号
>>> print(prefix"thon")
  File "<stdin>", line 1
    print(prefix"thon")
                ^
SyntaxError: invalid syntax
# 使用加号
>>> print(prefix+"thon")
python
```


# 索引


字符串支持索引访问，索引也称为下标。第一个字符的索引是0，最后一个字符的索引是**字符串的长度 - 1**

## 正数索引

```python
>>> "python"[0]
'p'
>>> "python"[1]
'y'
>>> "python"[2]
't'
>>>
```

```python
>>> str1 = "python"
>>> str1[0]
'p'
>>> str1[1]
'y'
>>> str1[2]
't'
>>>
```

## 负数索引

索引还能搞负数的？那咱们试一下

```python
>>> "python"[-1]
'n'
>>> "python"[-2]
'o'
>>> "python"[-3]
'h'
```

**原来负数是从尾到头。。。**

## 字符串切片

字符串：没人为我花生吗？都要把我切片了~

作者：嘿嘿嘿~~~~~~咱们开始吧~

要注意的第一点：**切片是左闭右开的，数学中表示为[0, 2)**,在python中 str1[0:2],表示从索引0开始，到1结束。或者按照官方的解释，**截取字符串的长度是索引之差**

要注意的第二点：**切片就算给了负数索引，也是从左到右切片**

```python
>>> "python"[0:2]
'py'
>>> "python"[-3:-1]
'ho'
```

参考官方文档给的解释

```python
+---+---+---+---+---+---+
 | P | y | t | h | o | n |
 +---+---+---+---+---+---+
 0   1   2   3   4   5   6
-6  -5  -4  -3  -2  -1
```

那么我只想截取一个范围呢

```python
>>> "python"[3:]
'hon'
>>>
>>> "python"[3:]
'hon'
>>> "python"[-3:]
'hon'
>>>
```

## 索引越界

先上代码

```python
>>> "python"[-10:]
'python'
>>> "python"[-20:]
'python'
>>> "python"[10:]
''
>>> "python"[10]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
>>>
```

原来我们用切片处理的时候，python会帮我们处理切片越界的问题。

但是直接使用索引读取某个字符，会报IndexError索引错误的错。


# 预告

字符串还拥有着很多的处理方法。暂且不表，咱们先了解清楚数据类型，后续再讲操作。

下一期列表~

------------------------------------------------------------------------

参考资料：
https://docs.python.org/zh-cn/3.11/tutorial/introduction.html#lists


