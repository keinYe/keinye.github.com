---
layout: post
title:  "Python 编程必不可少的测试框架「unittest 篇」"
date:   2020-01-09 20:57:22 +0800
author: keinYe
categories: [ Flask ]
image: assets/images/3.jpg
beforetoc: "."
toc: true
tags: [python, 测试, RESTful, Flask]
---

> 没有经过测试的代码是不可靠的代码。

unittest 是一个单元测试框架，单元测试完成对一个模块、一个类或一个函数的运行结果进行检验的测试工作。单元测试是对一个程序最基础的组成部分进行正确性验证，只有所有的单元测试不存在问题才能保证整体程序的正确性。

==一个优秀的程序不仅能够写出优秀的功能代码，也要能够写一手优秀的测试代码。==

> 测试驱动开发：要求在开始编写功能代码之前，先编写测试代码，然后编写能够通过测试的代码，通过测试来推动整个开发过程进行。

通过编写测试代码，不仅使当前的代码设计能够满足预期的功能需要，在将来对代码进行修改时，也能够保证修改后代码的输出结果是正确的。

unittest 是 python 自带的单元测试框架，test fixture「测试框架」、test case「测试用例」、test suite「测试集合」、test runner「测试运行器」是 unittest 的四个核心概念。

- **test fixture**：测试框架，在测试开始前进行一些必要的准备工作，或在测试结束时进行相关的清理工作。
- **test case**：测试用例，作为一个单独的测试单元，提供实际的测试逻辑，检测特定的输入所对应的输出结果是否与预期一致。
- **test suite**：测试集合，将多个测试用例组合起来，统一进行测试。既可以由多个测试用例组成，也可以有多个测试集合组成，还可以是测试用例和测试集合共同组成。
- **test runner**：测试运行器，用于执行测试和输出测试结果。

---

编写测试代码时，我们需要编写一个继承自 unittest.TestCase 的测试类，在该类中以 test 开头的方法就是测试方便，在测试过程中会被执行，不以 test 开头的方法在测试时会被跳过。在测试类中有两个特殊的方法 setUp 和 tearDown，这两个方法分别完成资源的创建和销毁，unittest 在调用测试方法之前会先执行 setUp 方法，在测试方法执行完成后会执行 tearDwon 方法，这样将资源的创建与销毁统一起来，不必在测试方法中编写重复的相同代码。

以下是一个测试 math 类中 fabs 函数和 isfinite 函数的单元测试，文件命名为```testMath.py```。
```python
import unittest
import math

class TestMath(unittest.TestCase):
    def setUp(self):
        print("setup Func ...")

    def test_abs(self):
        self.assertEqual(3, math.fabs(1 - 4))
        self.assertNotEqual(1, math.fabs(1 - 2))

    def test_isfinite(self):
        self.assertTrue(math.isfinite(456789))
        self.assertFalse(math.isfinite(float('inf')))
        self.assertFalse(math.isfinite(float('nan')))

    def tearDown(self):
        print("tearDown Func ...")

if __name__ == '__main__':
    unittest.main()
```
可以通过以下任意一个命令来运行该单元测试
```shell
python testMath.py
或
python -m unittest testMath
```

> 如果单元测试文件中未实现上面的最后两行代码，则只能使用 ```python -m unittest testMath``` 来启动测试。

该单元测试的输出结果如下：
```shell
(.venv) ➜  study python -m unittest testMath
setup Func ...
tearDown Func ...
Fsetup Func ...
tearDown Func ...
.
======================================================================
FAIL: test_abs (testMath.TestMath)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/path/testMath.py", line 12, in test_abs
    self.assertNotEqual(1, math.fabs(1 - 2))
AssertionError: 1 == 1.0

----------------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (failures=1)
```

从以上测试结果中可以看出 setUp 和 tearDown 函数分别被执行了两次，共进行了两个单元测试，其中有一个出现了错误，在错误提示信息中有错误的语句，错误的位置，以及错误出现的原因。我们共有两个单元测试，因此需要进行两个资源的创建和释放，所以 setUp 和 taerDown 函数各被执行了两次。在每个单元测试运行之前均进行了资源的创建「setUp 函数被执行」，在单元测试运行之后均进行了资源的释放「tearDown 函数被执行」。

---

unittest 不仅能够实现对基本函数的测试，同样还能够对复杂的应用进行测试，接下来我们共同来看下如何使用 unittest 来测试 Flask 应用的代码。
```python

import unittest
import os
from server import create_app
from server.module import db
from server.models.user import User, Permission
import tempfile
import json


DEFAULT_USERNAME = 'test'
DEFAULT_PASSWORD = 'test'

class ServerTestUser(unittest.TestCase):
    def setUp(self):
        self.db_fd, self.db_file= tempfile.mkstemp()
        app = create_app()
        app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + self.db_file
        self.app = app
        self.clinet = app.test_client()
        with app.app_context():
            db.drop_all()
            db.create_all()
            user = User.create(
                name=DEFAULT_USERNAME,
                password=DEFAULT_PASSWORD,
                permission=Permission.ADMINISTRATOR,
                active=True)
            self.headers = self.login()


    def login(self):
        rv = self.clinet.post('/api/v01/user/login',
                               data=json.dumps(dict(user_name='test', password='test')),
                               content_type='application/json')
        data = json.loads(rv.data)
        token = data['token']
        headers = {"Authorization":"Bearer "+token, 'Content-Type': 'application/json'}
        return headers

    def test_login(self):
        rv = self.clinet.post('/api/v01/user/login',
                               data=json.dumps(dict(user_name=DEFAULT_PASSWORD, password=DEFAULT_PASSWORD)),
                               content_type='application/json')
        data = json.loads(rv.data)
        self.assertEqual(rv.status_code, 200)
        self.assertEqual(data['status'], 1)
        self.assertEqual(data['name'], 'test')
        self.assertIsNotNone(data['token'])
        self.assertIsNotNone(data['admin'])
        self.assertIsNotNone(data['expire'])

    def test_add_user(self):

        rv = self.clinet.post('/api/v01/user',
                               data=json.dumps(dict(user_name='123', password='123', admin=False)),
                               headers=self.headers)
        data = json.loads(rv.data)
        self.assertEqual(data['status'], 1)



    def tearDown(self):
        with self.app.app_context():
            db.session.remove()
            db.drop_all()
        os.close(self.db_fd)
        os.unlink(self.db_file)
```
以上代码完成了使用 RESTful API 登录以及登录后添加新用户的 API 的测试。

在 setUp 函数中创建了 Flask 对象，通过 tempfile 创建临时文件用于数据存储，在 Flask 的运行环境中生成数据表、加入默认的用户，同时获取登录 Token 用户后面的 API 测试认证。在 tearDwon 函数中完成测试后的资源清理工作，删除数据表并删除创建的临时文件。

在 test_login 和 test_add_user 函数中完成了对登录 API 和用户添加 API 的测试，并检测返回结果的正确性。

使用命令 ```python -m unittest discover -v -s tests``` 启动测试，测试结果如下：
```shell
(.venv) ➜  server git:(master) ✗ python -m unittest discover -v -s tests
test_add_user (test_user.ServerTestUser) ... ok
test_login (test_user.ServerTestUser) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.741s

OK
```

公众号发送 Flask 获取相关源码！
