---
layout: post
title:  "python 模块"
date:   2020-04-08 07:35:22 +0800
author: keinYe
categories: [ Python ]
image: assets/images/7.jpg
beforetoc: "."
toc: true
rating: 4
tags: [python, module, Flask]
---

> 软件模块（Software Module）是一套一致而互相有紧密关连的软件组织。包含了程序和数据结构两个部分。
>软件模块是现代软件开发往往利用模块作合成的单位。模块的接口表达了由该模块提供的功能和调用它时所需的元素。模块是可能分开地被编写的单位，能允许广泛人员同时协作、编写及研究不同的模块。

以上是维基百科中对软件模块的解释。使用模块大大提高了代码的可维护性及可重复使用，给软件行业的分工协作提供了更多的可能。

在 Python 中模块的组成形式多种多样，一个包含函数与变量、以 ```.py``` 为后缀的文件的即可看做是一个模块；也可以使用 C 语言来编写模块，在编译以后通过 Python 解释器来在 Python 代码中使用。

## 使用模块
在使用模块之前需要先将模块导入到你的文件，在 Python 中通过 ```import``` 或 ```from...import``` 来导入一个模块。Python 总是导入搜索到的第一个匹配模块，在运行过程中 Python 会在 ```sys.path``` 变量所提供的目录中搜索模块。

> 默认情况下，Python 会搜索当前目录、内置模块和安装的第三方模块，搜索的路径包含在 ```sys.path``` 中。 

下面以导入 ```sys``` 模块为例，介绍如何使用 ```import``` 导入并使用该模块：
```python
import sys

print(sys.path)
print(sys.version)
```
执行以上代码将输出以下内容，由于系统的差异输出内容可能会有一定的差异。
```shell
['/Users/keinYe/Work/python/study', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python37.zip', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7', '/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload', '/Users/keinYe/Library/Python/3.7/lib/python/site-packages', '/usr/local/lib/python3.7/site-packages', '/usr/local/opt/opencv/lib/python3.7/site-packages']
3.7.5 (default, Nov  1 2019, 02:16:32) 
[Clang 11.0.0 (clang-1100.0.33.8)]
```
在使用 ```import``` 导入一个模块后，在文件中将获取到一个和模块名称同名的变量，通过该变量我们可以访问该模块的所有功能，访问模块的功能通过点号 ```.``` 来实现。

由于在文件中我们将模块的名称作为变量来使用，那么不同的模块具有相同的名称，那么我们是否就不能在一个文件中同时使用者两个模块了呢？答案是否定的，Python 提供了 ```as``` 关键字允许你在导入模块时将模块进行重命名。
```python
import sys as default

print(default.version)
```
当使用 ```as``` 重命名后，模块原有的名称所生成的编在文件中将不存在，使用该变量 Python 解释器将抛出 ```NameError``` 错误。

如果你只需要使用模块下的单个功能，使用 ```import``` 导入模块，没事使用该功能时都要输入模块的名称，显然有些繁琐，此时可以使用 ```from...import``` 导入模块的单个功能来使用。
```python
from sys import version
print(version)
```
> 使用 ```from...import``` 导入模块可能会造成命名冲突，同时也会降低代码的可读性，因此除非必要不推荐使用 ```from...import``` 来导入模块。

## 编写一个模块
在 Python 中编写以个模块非常简单，因为一个以 ```.py``` 为后缀的文件就是一个模块，文件的名称就是模块的名称。
```python
#!/usr/bin/env python3
#-*- coding:utf-8 -*-

'''
    first module
'''
__author__ = 'keinYe'
__version__ = '0.1'

def say():
    print('Hi, this is my fisrt python module.')
```
以上代码的第一行注释标识了该文件的运行环境，它让该文件可以在 Unix/Linux/Mac 等系统上直接运行。第二行注释表示该文件使用标准 ```utf-8``` 编码进行存储。

接下来的字符串是该文件的注释，在 Python 文件中任何第一个出现的未被赋值给变量的字符串都被作为文件的注释。```__author__``` 标识文件的作者，```__verson__``` 标识代码的版本。

将该内容保存为 ```mymodule.py``` 文件，我们就可以在同路径文件中导入该模块并使用它。
```python
import mymodule

mymodule.say()
```
> 如果使用的文件和模块不再同一个文件夹内，确保 ```sys.path``` 包含模块的路径。

## 使用第三方模块
在 Python 中第三方模块的管理，通过包管理工具 ```pip``` 来进行。在使用第三方模块之前需要先安装第三方模块，安装模块要确定模块名称，然后通过 ```pip install 模块名称``` 的命令来进行安装，安装完成以后就可以通过 ```import``` 或 ```from...import``` 导入模块来使用。

通常情况下一个开发项目会使用到非常多的模块，更别说每个模块还有很多的版本，记住每一个模块和所使用的版本并安装他们是一件困难的事情，此时我们可以将模块写入一个文件中，在文件中标识出每个模块的名称和他们的版本，然后通过该文件来一次性的安装所有的模块，避免安装时出现错误。

下面是一个以 ```requirements.txt``` 的文件，该文件标识出了项目中所用到的模块和他们的最低版本。
```
click>=6.7
Flask>=1.0.0
Flask-Cors>=3.0.7
Flask-HTTPAuth>=3.3.0
Flask-Login>=0.4
Flask-Migrate>=2.0.4
Flask-Redis>=0.4.0
Flask-SQLAlchemy>=2.3.2
flask-restful>=0.3
itsdangerous>=0.24
Jinja2>=2.10
PyMySQL>=0.9.2
pluggy>=0.6.0
pytz>=2018.4
SQLAlchemy-Utils>=0.33.3
Werkzeug>=0.14.1
pyecharts>=1.0.0
pytest>=5.3.0
```
然后我们可以通过 ```pip install -r requirements.text``` 来安装该文件中的所有模块。

## 包
包是一种按目录来组织模块的方式，它能够方便的分层组织模块。包的引入解决了模块重名的问题，因为只要包目录的名称不同，包内所有模块都不会与其他人的模块冲突。一个简单的包组织方式如下：
```
mymodule
├─ __init__.py
├─ abc.py
└─ xyz.py
```
使用包来组织模块以后，使用包内的任何模块都要加上包名，此时 ```abc``` 模块的引用变成了 ```mymoduel.abc```。

每个包目录下都必须有一个 ```__init__.py``` 的文件，否则 Python 会把该目录当做一个普通目录，而不是一个包。```__init__.py``` 可以是空文件，也可以有代码，因为它本身就是一个模块，它的模块名就是包名。

## 作用域
在 python 中不存在 ```prative``` 和 ```public``` 等这样的关键字来标识变量和函数的作用域，在 Python 中通过 ```_``` 前缀来标识变量和函数的作用域。

正常可在模块外部访问的变量和函数，正常命名即可「不要在名称前加任何的 ```_``` 前缀」。

```__xx__``` 在名称前后均有两个 ```_``` 的变量称为特殊变量，可以引用但是有特俗的用途，我们自己的变量一般不要这样命名。

```_xx``` 和 ```__xx``` 只有在名称前有 ```_``` 的变量和函数称为私有变量和函数，不应该被直接引用，

> Python 本身并没有私有变量和函数的概念，解释器本身并能阻止你访问 ```_xx``` 和 ```__xx``` 这样的变量和函数，这只是一种约定。