---
layout: post
title:  "外行学 python 爬虫 第二篇 获取内容"
date:   2019-05-02 21:37:22 +0800
tags: [python, 外行, 爬虫]
author: keinYe
categories: [ python ]
image: assets/images/2.jpg
beforetoc: "一个无法获取内容的爬虫不是一个真正的爬虫，爬虫的首要目标是从网络上获取内容."
toc: true
---
一个无法获取内容的爬虫不是一个真正的爬虫，爬虫的首要目标是从网络上获取内容。目前我们所看到的网页都是通过超文本传输协议「英语：HyperText Transfer Protocol，缩写：HTTP」在服务器和客户端之间进行数据交换。

从网站上获取内容实际上就是一个 HTTP 的通信过程，服务器还是那个服务器，只是客户端从浏览器换成了我们的爬虫程序。爬虫程序实现的就是浏览器的功能，有很多时候还需要模仿浏览器的行为「比如登录、获取 cookie 等等」才能够从服务器获取我们需要的数据。

HTTP 通信过程可以简单的分为两个部分请求和应答。请求有客户端发起、服务器在接收到客户端的请求后，组织应答数据并将数据通过 HTTP 协议发送给客户端，请求和应答组成了一个完整的网络通信过程。

在 HTTP 协议中请求分为GET、PUT、POST、DELETE 等几种，GET向指定的资源发出“显示”请求，以从服务器中获取数据；PUT向指定资源位置上传其最新内容；POST向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）；DELETE请求服务器删除所标识的资源。GET 方法在爬虫程序中是最主要也是最长用的方法。

在 python 中可以通过内置的 urllib 库来获取网站内容，可以通过 Selenium 库来模拟浏览器的行为。urllib 是 python 标准库中专门用于网络请求的库，强烈建议初学者使用 urllib 来实现网络请求，urllib 可以完成当前所遇到任何问题。

使用 urllib 仅仅 2 行代码即可实现网络内容的读取：
```python
response = request.urlopen(url, timeout=10)
html = response.read()
```
在以上代码中 html 即从网络上获取的 url 的网页内容。

对于 urllib 的使用方法在[初识 Python 网络请求库 urllib](https://mp.weixin.qq.com/s/aX_34hPMX-waJQw5fsA_og)中已经进行过介绍，这里就不再详细介绍了。
