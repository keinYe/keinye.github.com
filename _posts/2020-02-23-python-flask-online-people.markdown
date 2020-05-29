---
layout: post
title:  "Python 库 xlwings 操作 Excel 文档"
date:   2020-03-02 20:57:22 +0800
author: keinYe
categories: [ Python ]
image: assets/images/5.jpg
beforetoc: "."
toc: true
rating: 4
tags: [python, redis, RESTful, Flask]
---

Python 中操作 Excel 的扩展库主要有：
- [***xlwings***](https://docs.xlwings.org/zh_CN/latest/quickstart.html)：在 GitHub 上获得了 1.6k 的 Star。 可结合 VBA 实现对 Excel 的编程。
- [***openpyxl***](https://www.osgeo.cn/openpyxl/tutorial.html)：代码托管在 Bitbucket 上 https://bitbucket.org/openpyxl/openpyxl/src/default/。 简单易用，功能广泛，还支持图表功能，但是对 VBA 支持不好。
- [***pandas***](http://pandas.pydata.org/)：在 GitHub 上获得了 23.8k 的 Star。主要用于数据处理。
- [***win32com***](http://pythonexcels.com/python-excel-mini-cookbook/
)：在 GitHub 上获得了 2.1k  的 Star。==仅支持 window 系统==
- [***xlsxwriter***](https://xlsxwriter.readthedocs.io/)：在 GitHub 上获得了 2.1k  的 Star。同 openpyxl 的功能相似，但是不能打开/修改已有文件。
- [***xlutils***](https://pypi.python.org/pypi/xlutils/)：仅支持 xls 文件及 excel 2003 及以前版本。

每个扩展库的功能都有其侧重点，根据所需要的功能，选择所需的扩展库即可。这里主要介绍通过 xlwings 对 Excel 文件进行操作。

选择 xlwings 的几个原因：
- 工作系统同时兼容 Windows 和 Mac ，支持 WPS 文件。
- 语法简单，基本上用过一次就会记住。
- 功能齐全，支持新建、打开、修改、保存等等。
- 可以调用已有的 VBA 程序（这个暂时我还没有用到）。

> 通过 ```pip install xlwings``` 来安装 xlwings，当前最新版本是 0.18.0

xlwings 的口号是："==让 Excel 跑的飞快=="，感觉有点意思。它是基于 BSD-licensed(伯克利软件发行版许可协议) 的Python库，它让Python和Excel之间的相互调用变得更加容易。

#### 新建文件
通过以下命令来新建一个 Excel 文件
```python
import xlwings as xw

wb = xw.Book() # or xw.books()
wb.save('test.xlsx')
```
或另一种方式
```python
import xlwings as xw

app = xw.App()

try:
    wb = app.books.add()
    wb.save('test.xlsx')
    wa.close()
except Exception as err:
    print(err)
app.quite()
```

#### 打开已有文件
通过以下命令来打开一个已有文件
```pyhon
import xlwings as xw

wb = xw.Book('test.xlsx') # ro xw.bookx('test.xlsx')
```
> 在上面的示例中使用的相对路径，你也可以使用绝对路径。
> 当在 windows 上使用时，注意 "\" 的转义问题，推荐直接在路径字符串上加 r 「r'g:\python\test.xlsx」
。

也可以使用另外一种方式
```python
import xlwings as xw

app = xw.App()
path = r'g:\python\test.xlsx
try:
    wb = app.books.open(path)
    wb.save()
    wa.close()
except Exception as err:
    print(err)
app.quite()
```

#### 读取并修改文件内容 
Excel 文件是由一个个工作表组成的，要想操作文件内容，需要先打开工作表「sheet」，打开 sheet 由以下几种方式：
```python
sheet = wb.sheets[0]       #下标定位打开第一个工作表
sheet = wb.sheets['test']  #打开名字为 test 的工作表
sheet = wb.sheets.active   #打开当前激活的工作表
```
工作表是由一个个单元格组成的，最终我们操作的是一个个单元格中的数据，接下来一块来看下单元的数据操作。
```python
sheet = wb.sheets[0]
sheet.range('A1').value = 1     #向 A1 单元格写入 1
print(sheet.range('A1').value)  #读取 A1 单元格中的内容
# 1.0
```
> 根据单元格里面存储的是数字、字符串、空白还是日期，返回的 python 对象类型分别是 float, unicode, None 或 datetime

前面操作的是单个单元格，接下来我们来操作一行或一列
```python
sheet.range('A1').value = [1, 2, 3, 4, 5]  #向 A1:E1 写入数据
print(sheet.range('A1:E1').value)          #读取 A1:E1 的数据
# [1.0, 2.0, 3.0, 4.0, 5.0]
```
```sheet.range('xx').value``` 默认按行写入数据，那么如果我们想要按列写入数据怎么办，按列写入数据有以下两种方法。
```python
sheet.range('A1').value = [[1],[2],[3],[4],[5]]
print(sheet.range('A1:A5').value) 
# [1.0, 2.0, 3.0, 4.0, 5.0]
# 或者使用下面的方法
sheet.range('A1').options(transpose=True).value = [6, 7, 8, 9, 10]
print(sheet.range('A1:A5').value) 
# [6.0, 7.0, 8.0, 9.0, 10.0]
```
大多数情况下，我们操作数据表写入的都是多行的数据，由上面的基础实际上多行数据就是一个二维的列表，一次写入和读取多行的方法如下：
```python
sheet.range('A1').value = [['Foo 1', 'Foo 2', 'Foo 3'], [10, 20, 30]]
print(sheet.range('A1:C2').value)
# [['Foo 1', 'Foo 2', 'Foo 3'], [10.0, 20.0, 30.0]]
print(sheet.range((1, 1), (2, 3)).value)
# [['Foo 1', 'Foo 2', 'Foo 3'], [10.0, 20.0, 30.0]]
```
但是 xlwings 还提供了另外一种更加方便的方式来操作一个区域块，通过 expand 或 options 中的 expand 参数，expand 使用的是当前已获取的区域对象，而 options 中的 expand 参数在条用时才计算区域对象，推荐使用 options 中的 expand 参数，是你可以在更改区域后及时获取区域的变化。下面的代码，可以清楚的表达两种方式的不同。
```python
sheet.range('A1').value = [[1,2], [3,4]]
rng1 = sheet.range('A1').expand('table') 
rng2 = sheet.range('A1').options(expand='table')
print(rng1.value)
# [[1.0, 2.0], [3.0, 4.0]]
print(rng2.value)
# [[1.0, 2.0], [3.0, 4.0]]
sheet.range('A3').value = [5, 6]
print(rng1.value)
# [[1.0, 2.0], [3.0, 4.0]]
print(rng2.value)
# [[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]]
```

以下是一个完整的例子：
```python
import xlwings as xw

wb = xw.Book()
sheet = wb.sheets[0]
sheet.range('A1').value = [1, 2, 3, 4, 5]
sheet.range('A2').value = ['a', 'b', 'c']
rng1 = sheet.range('a1').expand('table')
val = rng1.value
print(val)

for i in val:
    if isinstance(i, list):
        for j in i:
            print('value=%s, type=%s' %(str(j), type(j)))
    else:
     print(i)

count = rng1.columns.count
sum = 0
value = sheet[0, :count].value
for i in sheet[0, :count].value:
    sum = sum + i;
value.append(sum)
sheet.range('A1').value = value
rng2 = sheet.range('A1').options(expand='table')
print(rng2.value)
print(rng1.value)

wb.save('test.xlsx')
wb.close()
```

运行以后的输入结果如下：
```shell
[[1.0, 2.0, 3.0, 4.0, 5.0], ['a', 'b', 'c', None, None]]
value=1.0, type=<class 'float'>
value=2.0, type=<class 'float'>
value=3.0, type=<class 'float'>
value=4.0, type=<class 'float'>
value=5.0, type=<class 'float'>
value=a, type=<class 'str'>
value=b, type=<class 'str'>
value=c, type=<class 'str'>
value=None, type=<class 'NoneType'>
value=None, type=<class 'NoneType'>
[[1.0, 2.0, 3.0, 4.0, 5.0, 15.0], ['a', 'b', 'c', None, None, None]]
[[1.0, 2.0, 3.0, 4.0, 5.0], ['a', 'b', 'c', None, None]]
```
程序运行完成后，在当前文件下将看到一个 test.xlsx 的 Excel 文件，文件内容如下：

#### 其他
- 清除单元格内容和格式 ```sheet.range('A1').clear()```
- 单元格的列标 ```sheet.range('A1').column```
- 单元格的行标 ```sheet.range('A1').row```
- 单元格的行高 ```sheet.range('A1').row_height```
- 单元格的列宽 ```sheet.range('A1').column_width```
- 列宽自适应 ```sheet.range('A1').columns.autofit()```
- 行高自适应 ```sheet.range('A1').rows.autofit()```
- 单元格背景色(RGB) ```sheet.range('A1').color = (34,139,34)```
- 清除单元格颜色 ```sheet.range('A1').color = None```
- 输入公式，相应单元格会出现计算结果 ```sheet.range('A1').formula='=SUM(A1:E1)```
- 获取单元格公式 ```sheet.range('A1').formula_array```
