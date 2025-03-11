---
title: 从零开始学Python（一）- 数据类型和变量
date: 2025-03-11 09:58:30
categories:
  - python
tags:
  - python
  - python basics  
---

从零开始学python基础，包括类型和变量，字符串和编码，list&tuple，条件判断，模式匹配，循环，dict&set

<more>

# print函数

可以用于向屏幕（控制台）输出制定的文字。

在我们安装好python以后，在控制台输入python --version，可以看到会打印出python的版本

```python
(base) PS C:\Users\hello_user> python --version
Python 3.9.1
```

直接输入python，会进入python的命令行模式，然后我们输入print("Hello python")，控制台输出Hello python字符串，最后输入exit()函数，就会退出python命令行模式

```python
(base) PS C:\Users\hello_user> python
Python 3.9.1 (default, Dec 11 2020, 09:29:25) [MSC v.1916 64 bit (AMD64)] :: Anaconda, Inc. on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello python")
Hello python
>>> exit()
(base) PS C:\Users\hello_user>
```

打印多行内容

```
>>> print('''line1
... line2
... line3''')
line1
line2
line3
```

# 数据类型和变量


## 数据类型

计算机程序可以处理各种数据，还可以处理图片，文本，音频，视频，网页等各种各样的数据，因此产生了不同的数据类型，使用数据类型去标记数据

### 数据类型-整数

整数包括正整数和负整数，例如1000，-1000, 0， -100等。我们可以使用十进制，十六进制表示整数。

```
# 十进制整数
a = 100
b = -100
# 十六进制
c = 0xa1
```

### 数据类型-浮点数

浮点数也可以称为小数，在科学计数法表示的时候，小数点的位置是可变的，于是也称为浮点数。

```
>>> 1.11*10**3 == 11.1 * 10**2
True
>>>
```

但是我们要注意一个问题，刚刚不是说小数点位置可变，但是值为何不一样

```
>>> 1.1*10**2 == 11 * 10
False
>>>
```

让我们打印一下经典问题0.1 + 0.2 == 0.3 #False

```
>>> 0.1 + 0.2
0.30000000000000004
>>>
```

整数和浮点数在计算机内部存储的方式是不同的，导致浮点数会有误差。扯远了，具体的要去了解计算机组成的知识。


### 数据类型-字符串

单引号或者双引号扩起来的文本

```
str1 = 'abc'
str2 = "def"
```

如果单引号本身也是字符的一部分,可以采用双引号包括单引号，或反过来。还可以使用\转义字符

```
str3 = "I'm python"
str4 = '"哇哇"'
str5 = "I \'m python"
```

如果过多的字符需要转义，可以使用r''表示内部不转义

```
>>> str6 = r"\\\\\/////"
>>> print(str6)
\\\\\///// # 完整打印
```

### 数据类型-布尔值

python中的布尔值有True和False，经常用于条件判断。

```
# 1 == 2会返回False
if 1 == 2: 
    print("1 不等于 2")
```

布尔值还可以用 and or not 运算

and：条件都为True，and运算才是True

```
if 1 == 1 and 2 == 2:
    print("全部条件通过")
```

or: 其中之一的条件为True，运算为True

```
if 1 == 1 and 3 == 2:
    print("全部部分通过")
```

not: 单目运算符，True会变False，反之也一样

```
if not True: 
    print("不通过") # 这里不会打印，因为not True变成False， if False 后面不会执行
```

### 空值

用None表示为空

```
str1 = None
```

## 变量

### 变量名

变量名必须是英文大小写，数字和_下划线组合，数字不能作为变量名开头

```
a = 1
b = 2
A = 10
A_a = 20
my_data = "I am a hero"
23_var = "sdf" # 报错，不可用数字开头
```


### 变量在计算机中的存储

```
str1 = "I like python"
```

计算机在内存创建I like python的字符串，再创建一个str1的变量指向字符串

```
str1 = "I like python"
str2 = str1
str1 = "I like php"
```

- 计算机在内存创建I like python的字符串,再创建一个str1的变量指向字符串
- 创建一个str2的变量指向字符串，这时候 str1 和 str2都指向了字符串，而不是str2指向str1
- 计算机在内存创建I like php字符串，str1再指向这个字符串

### 常量

不能变的变量，习惯性使用大写表示，但是变量的本身，是可以指向其他数据的

```
PI = 3.14
# 我是可以改成3.15的
PI = 3.15
```

