---
layout: post
title:  "python 反射"
date:   2020-04-09 07:35:22 +0800
categories: Python
tags: [python, module, Flask, 反射]
---


> 在计算机学中，反射（英语：reflection）是指计算机程序在运行时（runtime）可以访问、检测和修改它本身状态或行为的一种能力。[1]用比喻来说，反射就是程序在运行的时候能够“观察”并且修改自己的行为。

以上是维基百科中对反射的解释。我的理解反射是在运行过程中，获取和修改未知对象的属性和方法的一种解决方案。

反射在一定程度上提供了灵活性和通用性，是程序在运行过程中根据实际需要执行额外的操作。同时反射降低了代码的可读性，为团队间的协作制造了一定的障碍。

反射广泛应用于软件测试中，在运行中创建和实例化对象，并操作属性和方法。

## Python 中的反射
Python 中反射的方法：
- ```hasattr(obj, name)```: 判断对象中是否有以 name 命名的属性或方法。
- ```getattr(obj, name)```: 获取对象中以 name 命名的属性或方法，如果是属性获取到将是属性的值，如果是方法获取到的是方法的实例。
- ```setattr(obj, name, value)```: 为对象设置一个以 value 为值以 name 为名的属性或方法。
- ```delattr(obj, name)```: 删除对象中以 name 为名称的属性或方法。
- ```dir([obj])```: 该方法返回对象大多数属性名的列表。

反射中方法的使用示例：
```python
class Test():
    def __init__(self, name):
        self.name = name

    def say(self):
        print('Hello ' + self.name)

test = Test("keinYe")
if hasattr(test, 'name'):
    print('name = %s' % getattr(test, 'name'))
    setattr(test, 'name', 'python')
    print('name = %s' % getattr(test, 'name'))
else:
    print("The attribute 'name' not exist in Test")

delattr(test, 'name')
if not hasattr(test, 'name'):
    print('name is deleted')
    setattr(test, 'name', 'python')
    print('name = %s' % getattr(test, 'name'))

attr = dir(test)
print(attr)
```
运行以上代码，你将得到以下结果：
```shell
name = keinYe
name = python
name is deleted
name = python
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'name', 'say']
```
在使用 ```dir``` 函数获取对象的属性和方法时，我们得到很多以 ```__``` 开头和结尾的属性，我们本身并没有定义这些属性，这些属性是 python 自动添加的，它们保存了对象的元数据，我们可以去读取这些属性，但是尽量不要修改他们，以免引起不必要的错误。

### 判断获取到的对象是否是方法/函数
通过反射获取到的对象有属性也有方法或函数，可以通过以下方法判断获取到的对象是否是方法或函数。
1. 判断是否具有 ```__call__``` 属性
   ```python
   add = lambda a, b:a+b
   if hasattr(add, '__call__'):
       print(add(1, 2))
   ```
2. 利用 ```callable``` 判断
   ```python
   if callable(add):
       print(add(1, 2))
   ```
3. 利用 ```inspect``` 模块中的 ```isfunction``` 函数判断
   ```python
   from inspect import isfunction
   if isfunction(add):
       print(add(1, 2))
   ```
   > 注意：```isfunction``` 函数无法判断类中的方法。
   
   Python的inspect模块包含了大量的与反射、元数据相关的工具函数。
   
4. 使用```types.MethodType``` 或 ```types.FunctionType``` 来判断。
   以下代码无效
   ```
   from inspect import isfunction
   import types

   class Test():
       def __init__(self, name):
           self.name = name

       def say(self):
           print('Hello ' + self.name)

   test = Test("keinYe")
   print(type(test.say))
   print(type(isfunction))
   print(type(print))
   if isinstance(isfunction, types.FunctionType):
       print(True)
   if isinstance(test.say, types.MethodType):
       print(True)
   if isinstance(print, types.BuiltinMethodType):
       print(True)   
   ```
   以上代码输出结果
   ```shell
   <class 'method'>
   <class 'function'>
   <class 'builtin_function_or_method'>
   True
   True
   True
   ```
   > 注意：```print``` 等 python 内置函数/方法，使用 ```type``` 函数获取到的类型将是 ```<class 'builtin_function_or_method'>```。
   
### 方法/函数的使用

当使用反射获取到方法或函数的对象后，可以直接在后加上括号进行访问「前提是该函数没有参数」。
```python
func = getattr(test, 'say')
if hasattr(func, '__call__'):
    func()
```

如果方法/函数有参数，那么以上调用方法将会报 ```TypeError```。那么我们如果来获取函数的参数呢，使用 ```inspect``` 模块的 ```getfullargspec``` 函数来获取参数列表。
```
func = getattr(test, 'say')
if hasattr(func, '__call__'):
    args = inspect.getfullargspec(func)
    print(args.args)
```
执行以上代码，将得到以下结果：
```shell
['self']
```
由于 ```say``` 是类方法，其第一个参数总是 ```self```，此时如果我们在 ```say``` 方法上新增一个参数，我们可以看到获取到的参数列表的变化。
```python
class Test():
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def say(self, time):
        print('Hello ' + self.name)

test = Test('keinYe', 20)

func = getattr(test, 'say')
if hasattr(func, '__call__'):
    args = inspect.getfullargspec(func)
    print(args.args)
```
此时我们将得到 ```say``` 方法的两个参数 ```self``` 和 ```time```。

## 反射的应用
在使用 python 进行网络通信时，不可避免的会使用到 json 格式，那么将一个类转换为 json 或将一个 json 转换为类对象，如果每次都手动将类属性转换为 json 数据，那么将是一个非常费力不讨好的工作，我们可以使用反射来实现，让每个数据类继承这些方法，即可实现一次编程到处使用。

以下是示例代码：
```
import json

class Base(object):
    def to_json(self):
        attrs = dir(self)
        json_dict = {}
        for attr in attrs:
            if attr.startswith('_') or attr.endswith('_'):
                continue
            attr_obj = getattr(self, attr)
            if hasattr(attr_obj, '__call__'):
                continue
            json_dict[attr] = attr_obj
        return json.dumps(json_dict)

    def to_object(self, json_str):
        json_dict = json.loads(json_str)
        attrs = dir(self)
        for attr in attrs:
            if attr.startswith('_') or attr.endswith('_'):
                continue
            if hasattr(attr, '__call__'):
                continue
            if attr in json_dict:
                setattr(self, attr, json_dict.get(attr))            

class Person(Base):
    def __init__(self, name=None, age=None):
        self.name = name
        self.age = age
    
    def say(self):
        print('name = %s, age = %s' % (self.name, self.age))

person1 = Person('keinYe', 25)
print(person1.to_json())
person2 = Person()
json_str = '{"name":"test", "age": 27}'
person2.to_object(json_str)
person2.say()
```
执行以上代码将得到以下结果：
```shell
{"age": 25, "name": "keinYe"}
name = None, age = None
name = test, age = 27
```