---
layout: post
title:  "Flask 统计在线人数"
date:   2020-02-23 20:57:22 +0800
author: keinYe
categories: [ Python ]
image: assets/images/6.jpg
beforetoc: "."
toc: true
rating: 4
tags: [python, redis, RESTful, Flask]
---

服务端完成以后，如果检验应用的效果呢，在线人数/客户端是一个不错的指标。但是客户端的连接通常是短连接「请求建立一次连接，请求完成连接即断开」，基于这种情况服务端需要在每次的客户端请求时记录当前的时间，以此来间接实现在线人数/客户端的统计「比如：5 分钟内过连接的客户端认为处于在线状态」。

在每次请求中记录下用户/客户端的 ID 及当前的时间，可以通过以下方式：
> 1. 使用字典直接存储在内存中。
> 2. 在用户数据表中存储最后连接的时间。
> 3. 使用 radis 来存储连接信息。

一个服务端总是会有很多的 API 接口，要统计每个连接的时间，我们总不能在每个 API 接口下都写一遍统计函数吧「这样也太不 python 了」，
python 的方式应该是在 flask_httpatuh 的 ```verify_token``` 和 ```verify_password``` 装饰器中进行处理。
```python
@token_auth.verify_token
def verify_token(token):
    g.current_user = None
    user = User.verify_auth_token(token, current_app)
    if not user or not user.isActive():
        return False
    g.current_user = user
    mark_online(g.current_user)
    return True

@basic_auth.verify_password
def verify_password(username, password):
    g.current_user = None
    user = User.query.filter_by(name = username).first()
    if not user or not user.check_password(password) or not user.isActive():
        return False
    g.current_user = user
    mark_online(g.current_user)
    return True
```

### 使用字典直接存储在内存中
使用字典来存储最后连接时间，直接将用户 id 作为 ```kye``` 将时间作为 ```value``` 存入字典中，获取在线人数时，直接遍历字典比较时间即可。

```
import time

_inline_user = {}

def make_inline(user):
    _inline_user[user.id] = int(time.time())

def get_online():
    count = 0;
    minutes = int(time.time()) - 5 * 60
    print(minutes)
    for _, values in _inline_user.items():
        if values > minutes:
            count = count + 1
    return count
```

### 在用户数据表中存储最后连接的时间
直接在当前数据表中新增一列用来存储最后连接时间，需要对用户表进行修改
```python
class User(BaseModel, UserMixin):
    __tablename__ = 'user'

    ...
    lastseen = db.Column(db.DateTime, default=datetime.datetime.utcnow, nullable=True)
    ...
```
然后在每次连接时将时间同步到数据库即可，在读取时直接检索符合条件的用户即可。
```
from datetime import datetime, timedelta
from pytz import UTC

def mark_online(user):
    user.lastseen = datetime.now(UTC)
    user.save()

def get_online():
    diff = datetime.now(UTC) - timedelta(5)
    return User.query.filter(User.lastseen >= diff).count()
```
使用数据库保存，还可以查看指定时间段内的在线人数「比如 3 天内，5 天内」，只需要更改相应的过滤条件。

### 使用 redis 来存储连接信息
使用 redis 来存储信息，需要在你的电脑「服务器」上已安装 redis。redis 相关内容请看 [https://redis.io/](https://redis.io/)。

Flask 中使用 redis 有方便的第三方库 flask_redis。可以直接通过 pip 命令进行安装。
```shell
pip install flask-redis
```
确保 redis 已经在本机正常启动，并在配置文件中配置 URL：```REDIS_URL = "redis://localhost:6379" ```。
```python
from flask_redis import FlaskRedis

redis_client = FlaskRedis()

def mark_online(user):
    user_id = str(user.id).encode('utf-8')
    now = int(time.time())
    expires = now + (5 * 60) + 10
    all_users_key = "online-users/%d" % (now // 60)
    user_key = "user-activity/%s" % user_id
    p = redis_client.pipeline()
    p.sadd(all_users_key, user_id)
    p.set(user_key, now)
    p.expireat(all_users_key, expires)
    p.expireat(user_key, expires)
    p.execute()

def get_online():
    current = int(time.time()) // 60
    minutes = range(5)
    users = redis_client.sunion(
        ["online-users/%d" % (current - x) for x in minutes]
    )

    return len([int(u.decode('utf-8')) for u in users])
```
> 注意：还需要通过 redis_client.init_app(app) 将 flask 对象传入 flask_redis 对象。


使用字典和 redis 是直接将输入在内容中存储，如果系统掉电或以外重启将丢失所有的数据，直接在数据中存储会增加数据库的压力，各有利弊看个人喜好。