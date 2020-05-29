---
layout: post
title:  "Python 命令行神器 Click"
date:   2019-10-26 20:57:22 +0800
author: keinYe
categories: [ Flask ]
image: assets/images/16.jpg
beforetoc: "."
toc: true
tags: [python, flask, Click]
---

Click 是一个用于快速创建命令行工具的 Python 支持库，Click 具有高度可配置性，使用非常少的代码就可以创造一个优雅的命令行工具，Click 使创建命令行工具变得快速而有趣。

实际上 Python 标准库提供了一个默认的命令行工具 Argparse，但是对于 Click 来说 Argparse 使用起来非常的繁琐和麻烦，大多数人都很少使用它。Argparse 对比与 Click 就像网页解析中使用的 re 和 BeautifulSoup。

Click 有三个非常重要的特性：
- 任意嵌套命令
- 自动生成帮助页面
- 支持在运行时延迟加载子命令

## 使用 Click 可以做什么
Click 为命令行的开发封装了大量的方法，开发者只需要专注于具体的功能开发即可完成各种命令行工具。
> 在编写你的命令行工具之前，请先安装 [Click](https://github.com/pallets/click)，你可以使用 ```pip instll click``` 来完成安装。

在开始之前，先使用 Hello World 来熟悉一下 Click 的组织方式:
```python
import click

@click.command()
@click.option('--count', default=1, help='number of greetings')
@click.argument('name')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```
测试运行结果如下：
```shell
(.venv) ➜  server python hello.py --help
Usage: hello.py [OPTIONS] NAME

  Simple program that greets NAME for a total of COUNT times.

Options:
  --count INTEGER  number of greetings
  --help           Show this message and exit.
(.venv) ➜  server python hello.py --count=3 Click
Hello Click!
Hello Click!
Hello Click!
(.venv) ➜  server python hello.py --count=3
Usage: hello.py [OPTIONS] NAME
Try "hello.py --help" for help.

Error: Missing argument "NAME".
(.venv) ➜  server python hello.py click
Hello click!
(.venv) ➜  server
```
Click 不仅将一个普通的函数转化为一个命令行工具，还为它提供了选项、参数等，并且自动生成了帮助信息。

Click 支持选项和参数两种类型的脚本参数，使用 option 装饰器来使相应的函数增加命令行选项，使用 argument 装饰器使相应的函数增加命令参数。在以上示例中 count 是选项，而 name 是参数。从运行结果上来看选项会出现在帮助信息中，参数不会出现在帮助信息中；在命令运行过程中参数如果为空则会出现运行错误，选项可以是空。

Click 用来协助你生成各种各样的 CLI 工具。它的设计初衷是为了能够将任意系统嵌套在一起，在我们使用的大多数命令中都具有很多的子命令，比如 Git、pip、npm 等等，其本身就是一个命令，其还有很多的子命令可以运行。使用 Click 你可以很方便的创建类似的嵌套命令。

Click 通过 group 装饰器来创建一个命令组，将一个复杂的命令行进行解耦，将不同的逻辑放在不同的命令中。下面是一个简单的示例：
```python
# -*- coding:utf-8 -*-
import click


@click.group()
def git():
    '''These are common Git commands used in various situations:'''
    pass

@git.command()
def clone():
    '''Clone a repository into a new directory'''
    click.echo('Clone a repository!')

@git.command()
def init():
    '''Create an empty Git repository or reinitialize an existing one'''
    click.echo('Create an empty Git repository!')

if __name__ == '__main__':
    git()
```
测试运行结果如下：
```shell
(.venv) ➜  server python hello.py
Usage: hello.py [OPTIONS] COMMAND [ARGS]...

  These are common Git commands used in various situations:

Options:
  --help  Show this message and exit.

Commands:
  clone  Clone a repository into a new directory
  init   Create an empty Git repository or reinitialize an existing one
(.venv) ➜  server python hello.py clone
Clone a repository!
(.venv) ➜  server python hello.py init
Create an empty Git repository!
(.venv) ➜  server python hello.py init --help
Usage: hello.py init [OPTIONS]

  Create an empty Git repository or reinitialize an existing one

Options:
  --help  Show this message and exit.
```

## 在 Flask 中使用 Click
Click 是 Flask 的团队 pallets 开发的优秀的开源项目，其本身就是为了支持 Flask microframework 生态系统的，因此在 Flask 上使用 Click 非常的简单和方便。之所以 Click 会用在很多其他非 Flask 的项目中，是因为 Click 是在是太好用了而已。

在上一篇 [使用 Flask 创建 RESTful 服务](https://mp.weixin.qq.com/s/kR6oiyYGDdhhpD9HGMy7ZA) 中，将数据库初始化和第一个用户的注册放在了 API 中，通过 RESTful API 来完成。今天使用 Click 来实现相同的功能。
```python
# -*- coding:utf-8 -*-

import click, logging
from server import create_app
from flask.cli import FlaskGroup, ScriptInfo
from server.module import db
from sqlalchemy_utils import database_exists, create_database
from server.models.user import User

logger = logging.getLogger(__name__)

class ServerGroup(FlaskGroup):
    def __init__(self, *args, **kwargs):
        super(ServerGroup, self).__init__(*args, **kwargs)

def make_app(script_info):
    config_file = getattr(script_info, "config_file", None)
    return create_app(config_file)

def set_config(ctx, param, value):
    """This will pass the config file to the create_app function."""
    ctx.ensure_object(ScriptInfo).config_file = value

@click.group(cls=ServerGroup, create_app=make_app, add_version_option=False,
             invoke_without_command=True)
@click.option("--config", expose_value=False, callback=set_config,
              required=False, is_flag=False, is_eager=True, metavar="CONFIG",
              help="using a path like '/path/to/xxxx.cfg'")
@click.pass_context
def server(ctx):
    """This is the commandline interface for server."""
    if ctx.invoked_subcommand is None:
        # show the help text instead of an error
        # when just '--config' option has been provided
        click.echo(ctx.get_help())

@server.command()
@click.option("--username", "-u", help="The username of the user.")
@click.option("--password", "-p", help="The password of the user.")
def initDB(username, password):
    '''Create a database and add the first user'''
    if database_exists(db.engine.url):
        click.echo('The database is exists!')
    else:
        create_database(db.engine.url)
    db.create_all(bind=None)

    user = User.create(
        name=username,
        password=password)
```
> Flask 中默认提供了 shell 和 run 两个命令，并创建了 FlaskGroup 的类用于 Click，当我们需要新增命令时只需要继承 FlaskGroup 即可。

使用 Click 的命令程序，最好是编写成使用 setuptools 分发的模块，下面我们在 Flask 应用中添加 setuptools 的支持。

在顶层目录中新增 setup.py 文件，文件内容如下：
```python
# -*- coding:utf-8 -*-
from setuptools import setup

setup(
    name='server',
    version='0.0.1',
    entry_points="""
        [console_scripts]
        server=server.cli:server
    """,
)
```
要测试 server 命令，首先需要安装当前命令行程序 ```pip install -e .``` 通过该命令即可完成安装。安装完成后你将看到以下结果。
```shell
(.venv) ➜  server server --help
Usage: server [OPTIONS] COMMAND [ARGS]...

  This is the commandline interface for server.

Options:
  --config CONFIG  using a path like '/path/to/xxxx.cfg'
  --help           Show this message and exit.

Commands:
  db      Perform database migrations.
  initdb  Create a database and add the first user
  routes  Show the routes for the app.
  run     Run a development server.
  shell   Run a shell in the app context.
```
在测试结果中我们可以看到 db、initdb、routes、run、shell 子命令，其中 routes、run、shell 是 Flask 自带的 Click 命令，db 是由 Flask-Migrate 模块引入的数据库管理软件，而 initdb 命令使今天我们使用 click 生成的命令。
