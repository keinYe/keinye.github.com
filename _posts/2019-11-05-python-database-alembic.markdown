---
layout: post
title:  "Python 数据库迁移工具 Alembic"
date:   2019-11-05 20:57:22 +0800
author: keinYe
categories: [ Flask ]
image: assets/images/17.jpg
beforetoc: "."
toc: true
tags: [python, flask, Alembic, SQLAlchemy]
---
Alembic 是一款轻量型的数据库迁移工具，它与 [SQLAlchemy](https://mp.weixin.qq.com/s/QF41i58djnn-Hb6n8vbjew) 一起共同为 Python 提供数据库管理与迁移支持。

## Alembic 的应用
Alembic 使用 SQLAlchemy 作为数据库引擎，为关系型数据提供创建、管理、更改和调用的管理脚本，协助开发和运维人员在系统上线后对数据库进行在线管理。

同任何 Python 扩展库一样，我们可以通过 pip 来快速的安装最新的稳定版 Alembic 扩展库 ```pip install alembic```。

### 创建 Alembic 迁移环境
在使用 Alembic 之前需要先建立一个 Alembic 脚本环境，通过在工程目录下输入  ```alembic init alembic``` 命令可以快速在应用程序中建立 Alembic 脚本环境，当在命令行看到以下输出时，表示 alembic 脚本环境创建完成。
```shell
(.venv) ➜  server alembic init alembic
  Creating directory /<path>/alembic ...  done
  Creating directory /<path>/versions ...  done
  Generating /<path>/alembic/script.py.mako ...  done
  Generating /<path>/alembic/env.py ...  done
  Generating /<path>/alembic/README ...  done
  Generating /<path>/alembic.ini ...  done
  Please edit configuration/connection/logging settings in '/Users/keinYe/Work/python/server/alembic.ini' before proceeding.
```
你可以通过 -t 选项来选择一个初始化的模板，Alembic 目前支持三个初始化模板「通过 ```alembic list_templates``` 可以查看支持的模板类型」，默认情况下使用的是通用模板，在大多数情况下使用通用模板即可。

生成的迁移脚本目录如下：
```shell
├── alembic
│   ├── README
│   ├── env.py
│   ├── script.py.mako
│   └── versions
```
- **alembic 目录**：迁移脚本的根目录，放置在应用程序的根目录下，可以设置为任意名称。在多数据库应用程序可以为每个数据库单独设置一个 Alembic 脚本目录。
- **README 文件**：说明文件，初始化完成时没有什么意义。
- **env.py 文件**：一个 python 文件，在调用 Alembic 命令时该脚本文件运行。
- **script.py.mako 文件**：是一个 mako 模板文件，用于生成新的迁移脚本文件。
- **versions 目录**：用于存放各个版本的迁移脚本。初始情况下为空目录，通过 ```revision``` 命令可以生成新的迁移脚本。

> alembic 会在你的应用程序的根目录下生成一个 ```alembic.ini``` 的配置文件，在开始任何的操作之前需要先修改该文件中的 ```sqlalchemy.url``` 指向你自己的数据库地址。


### 生成迁移脚本
当 Alembic 配置环境创建完成后，可以通过 Alembic 的子命令 revision 来生成新的迁移脚本。
```shell
(.venv) ➜  server alembic revision -m "first create script"
  Generating /<path>/alembic/versions/4d1cbe22bfc6_first_
  create_script.py ...  done
```
初始的迁移脚本中并没有实际有效的内容，相当于一个空白的模板文件「增加了版本信息」。如果对整改工程的数据表进行修改后，再次运行 revision 子命令可以看到新生成的脚本文件中的内容增加了我们对数据表的改变内容。
```shell
(.venv) ➜  server alembic revision -m "add create date in user table"
  Generating /<path>/alembic/versions/fc540ca3e448_add_create_date_in
  _user_table.py ...  done
```
此时在 alembic 文件夹中可以看到以下文件:
```shell
alembic
├── README
├── env.py
├── script.py.mako
└── versions
    ├── 3602707b314b_first_create_script.py
    └── eac6fb06ced5_add_create_date_in_user_table.py
```
可以看到在 versions 目录中生成了两个迁移脚本文件，但是此时的迁移脚本文件中没有任何的有效代码，文件内容如下：
```python
"""add create date in user table
Revision ID: eac6fb06ced5
Revises: 3602707b314b
Create Date: 2019-11-03 06:54:23.862575
"""
from alembic import op
import sqlalchemy as sa
# revision identifiers, used by Alembic.
revision = 'eac6fb06ced5'
down_revision = '3602707b314b'
branch_labels = None
depends_on = None
def upgrade():
    pass
def downgrade():
    pass
```
在该文件中制定了当前版本号 ```revision``` 和父版本号 ```down_revision``` ，以及相应的升级操作函数 ```upgrade``` 和降级操作函数 ```dwongrade```。在 ```upgrade``` 和 ```dwongrade``` 函数中通过相应的 API 来操作 op 和 sa 对象来完成对数据库的修改，以下代码完成了在数据库中新增一个 account 数据表的功能。
```python
def upgrade():
    op.create_table(
        'account',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('description', sa.Unicode(200)),
    )
def downgrade():
    op.drop_table('account')
```
每次编写完代码还需要手动编写迁移脚本这并不是程序员所需要的，幸运的是 Alembic 的开发者为程序员提供了更美好的操作「自动生成迁移脚本」。自动生成迁移脚本无需考虑数据库相关操作，只需完成 ROM 中相关类的编写即可，通过 Alembic 命令即可在数据库中自动完成数据表的生成和更新。在 Alembic 中通过 revision 子命令的 --autogrenerate 选项参数来生成自动迁移脚本。

在使用自动生成命令之前，需要在 env.py 文件中修改 target_metadata 配置使其指向应用程序中的元数据对象。
```python
# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
# target_metadata = None
from server.module import metadata
target_metadata = metadata
```

在 user 数据表中新增 create_date 数据列，然后使用自动生成迁移脚本命令，查看我们的配置是否完成。运行命令后可以看到以下信息：
```shell
(.venv) ➜  server alembic revision --autogenerate -m 'add create date in user table'
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added column 'user.create_date'
  Generating /<path>/alembic/versions/476cb25a3138_add_create_date_in_user_table.py ...  done
```
生成的迁移脚本文件内容如下：
```python
def upgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.add_column('user', sa.Column('create_date', sa.DateTime(), nullable=True))
    # ### end Alembic commands ###

def downgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_column('user', 'create_date')
    # ### end Alembic commands ###
```


在自动生成迁移脚本的过程中你可能会遇到以下错误
```shell
(.venv) ➜  server alembic revision --autogenerate
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
ERROR [alembic.util.messaging] Target database is not up to date.
  FAILED: Target database is not up to date.
```
出现该错误的原因是没有使用 Alembic 更新数据库，如果你没有手动创建数据表可以使用 ```alembic upgrade head``` 命令消除该错误，如果你已经通过命令行或其他方式创建了数据表，可以使用 ```alembic stamp head``` 命令来设置 Alembic 的状态。

### 变更数据库
Alembic 最重要的功能是自动完成数据库的迁移「变更」，所做的配置以及生成的脚本文件都是为数据的迁移做准备的，数据库的迁移主要用到 ```upgrade``` 和 ```downgrade``` 子命令。

数据看的变更主要用到以下命令：
- **```alembic upgrade head```**：将数据库升级到最新版本。
- **```alembic downgrade base```**：将数据库降级到最初版本。
- **```alembic upgrade <version>```**：将数据库升级到指定版本。
- **```alembic downgrade <version>```**：将数据库降级到指定版本。
- **```alembic upgrade +2```**：相对升级，将数据库升级到当前版本后的两个版本。
- **```alembic downgrade +2```**：相对降级，将数据库降级到当前版本前的两个版本。

以上所有的升降级方式都是在线方式实时更新数据库文件，实际环境中总会存在一些环境无法在线升级，Alembic 提供了生成 SQL 脚本的形式，已提供离线升降级的功能。
```shell
alembic upgrade <version> --sql > migration.sql
```
或
```shell
alembic downgrade <version> --sql > migration.sql
```

随着项目的进展，项目下不可避免的会生成很多版本的迁移脚本，此时可以使用 ```current``` 来查看线上数据库处于什么版本，也可以通过 ```history``` 来查看项目目录中的迁移脚本信息。
```shell
(.venv) ➜  server alembic current
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
71fe19b20211
(.venv) ➜  server alembic history --verbose
Rev: 476cb25a3138 (head)
Parent: 71fe19b20211
Path: /<path>/alembic/versions/476cb25a3138_add_create_date_in_user_table.py

    add create date in user table

    Revision ID: 476cb25a3138
    Revises: 71fe19b20211
    Create Date: 2019-11-03 07:20:23.594777

Rev: 71fe19b20211
Parent: <base>
Path: /<path>/alembic/versions/71fe19b20211_crate_user_table.py

    crate user table

    Revision ID: 71fe19b20211
    Revises:
    Create Date: 2019-11-03 07:19:03.221662
```

## 在 Flask 中使用 Alembic
在 Flask 可以通过 Flask-Migrate 快速集成 Alembic 的支持。

> Flask-Migrate 是使用 Alembic 处理 Flask 应用中数据库「使用 SQLAlchemy ORM」迁移的扩展库。其内置了 Click 命令行程序，在 Flask 上可直接使用命令行工具进行数据库的迁移。关于 Click 的使用请参考 [Python 命令行神器 Click](https://mp.weixin.qq.com/s/YQduuNZ1Grs2NeblazCt9g)。

Flask-Migrate 支持使用 pip 进行安装：
```shell
pip install flask-migrate
```
Flask-Migrate 安装完成后，会在 Flask 应用程序的命令下自动生成一个 db 子命令，我们通过该命令即可完成数据的操作。
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
(.venv) ➜  server server db --help
Usage: server db [OPTIONS] COMMAND [ARGS]...

  Perform database migrations.

Options:
  --help  Show this message and exit.

Commands:
  branches   Show current branch points
  current    Display the current revision for each database.
  downgrade  Revert to a previous version
  edit       Edit a revision file
  heads      Show current available heads in the script directory
  history    List changeset scripts in chronological order.
  init       Creates a new migration repository.
  merge      Merge two revisions together, creating a new revision file
  migrate    Autogenerate a new revision file (Alias for 'revision...
  revision   Create a new revision file.
  show       Show the revision denoted by the given symbol.
  stamp      'stamp' the revision table with the given revision; don't run...
  upgrade    Upgrade to a later version
```
Flask-Migrate 的用法和 Alembic 类似，只是将 alembic 换成了你的应用名称「或 flask」+ db 的方式。
