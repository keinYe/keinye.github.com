---
layout: post
title:  "python 装饰器"
date:   2020-04-10 07:50:22 +0800
author: keinYe
categories: [ Python ]
image: assets/images/9.jpg
beforetoc: "."
toc: true
rating: 4
tags: [python, module, Flask, 装饰器]
---

装饰器是应用包装函数的快捷方式。装饰器「decorator」能够在代码运行过程中动态给函数增加功能，或者在不修改他人代码的情况下对其增加新的功能。

## 一个简单的装饰器

Python 中的一切皆为对象，函数也不例外。既然函数时对象那么他就可以被赋值给对象，所以通过变量也能够调用函数；函数既然可以是变量，那么他就可以从函数中返回「Python 允许在函数中定义函数」，这两点是装饰器的基础。
> 装饰器本身是一个返回函数的高阶函数。Python 使用 @ 将一个装饰器附加到函数上。

```python
def new_decorator(func):
    def wrapper(*args, **kw):
        print('executing %s()' % func.__name__)
        return func(*args, **kw)
    return wrapper

@new_decorator
def test():
    print('decorator test')

test()
print(test.__name__)
```
在以上代码中我们定义了一个装饰器 new_decorator，它接收一个函数作为参数，在其内部定义了 wrapper 的函数，它的参数定义是 (*args, **kw) , 因此允许接收任意参数，这样在使用该装饰器的函数上将可以使用任意的参数。在 Python 中每个函数都有一个 ```__name__``` 属性可以访问当前函数的名称，以上代码的运行结果如下：
```shell
executing test()
decorator test
wrapper
```
装饰器被完美的添加在了 test 函数上。装饰器实现的功能实际上和我们使用 new_decorator(test) 的结果是相同的，Python 帮帮我们做了相同的事，因此我们得以使用原始函数获取到相同的结果。

从结果中也存在一个不正常的现象，那就是 test 函数的 ```__name__``` 属性值被修改成了 wrapper，这显然不是我们想看到的结果。这个问题我们可以通过在 wrapper 函数上使用 wraps 装饰器来解决。
> wraps 装饰器是 functools 包内的一个函数。

## 带参数的装饰器
前面我们说装饰器本身就是一个以函数为参数，返回函数变量的函数。那么函数可以有参数，装饰器应该也可以有，事实上确实如此。我们可以通过让装饰器返回一个装饰器来实现这个功能，我们在签名的代码的基础上做一个修改。
```
from functools import wraps

def new_decorator(text):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kw):
            print('text=%s func_name=%s()' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

@new_decorator('log')
def test():
    print('decorator test')

test()
print(test.__name__)
```
从代码可以看出，带参数的装饰器实际上就是一个嵌套函数，如果用函数调用的方式来实现装饰器效果，它看起来是这个样子 ```new_decorator('log')(test)```。运行以上代码你将看到如下结果：
```shell
text=log func_name=test()
decorator test
test
```

## 装饰器的实际应用
装饰器常用的两个场合是认证和日志。在使用 flask_httpauth 来实现 Flask 的 API 认证的过程中，我们是通过 verify_password 装饰器将密码校验的方法带入了 flask_httpauth，在需要进行登录保护的 API 上附加 login_required 来进行密码的校验工作。这样我们就无需在每个 需要校验的 API 上实现密码校验的函数。

在这里实现一个自定义的秘钥校验的装饰，为 API 增加一个秘钥校验的功能。
```python
class Secret(object):
    def __init__(self):
        self.verify_key_callback = None
        self.verify_error_callback = None

    def verify_key(self, f):
        self.verify_key_callback = f
        return f

    def error_handler(self, f):
        @wraps(f)
        def decorated(*args, **kwargs):
            res = f(*args, **kwargs)
            res = make_response(res)
            return res
        self.verify_error_callback = decorated
        return decorated

    def secret_required(self, f):
        @wraps(f)
        def decorated(*args, **kwargs):
            if request.method != 'OPTIONS':
                key = request.values.get('key', None)
                if key is None or not self.verify_key_callback(key):
                    return self.verify_error_callback(*args, **kwargs)
            return f(*args, **kwargs)
        return decorated
```
以上类定义三个装饰器函数 verify_key 直接返回了函数本身，通过该函数实现对秘钥的校验。error_handler 定义了一个校验错误时的装饰器，通过该函数实现出错时的应答。secret_required 该装饰器用在需要进行秘钥校验的 API 上，自动在当前请求参数上获取 key，使用 verify_key 函数进行校验，在依据校验的结果执行错误应答或正常应答。

verify_key 和 error_handler 实现如下：
```python
@secret_verify.verify_key
def verify_key(key):
    g.current_machine = Machine.verify_secret_key(key)
    if g.current_machine:
        return True
    return False

@secret_verify.error_handler
def secret_error_handler():
    return result.create_response(result.KEY_ERROR)
```
至于 secret_required 函数只需要正常附加到 API 函数上即可。
