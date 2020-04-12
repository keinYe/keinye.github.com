---
layout: post
title:  "Flask 使用 Redis 存储动态数据"
date:   2020-04-12 21:34:22 +0800
categories: Python
tags: [python, Redis, Flask]
---

```Redis``` 是一种 nosql 数据库，它的数据是保存在内存中的，因此其具有很快的存取速度；```Redis``` 通过定期将数据同步至磁盘来实现数据持久化。

使用场景：
- 登录回话存储。
- 排行榜/计数器，比如文章阅读数、点赞数。
- 作为消息队列。
- 当前在线人数统计。
- 常用数据的缓存，减少数据库访问压力。

## Redis 安装

> Redis 安装在 debian 系统上进行验证。

在 Debian 上安装 Redis 使用以下命令
```shell
apt update
apt install reids-server
```

修改 ```/etc/redis/redis.conf``` 文件中的 ```supervised``` 值为 ```systemd```。

使用以下命令重启 redis 以使配置生效。
```shell
systemctl restart redis
```

使用 ```systemctl stauts redis``` 命令来检测 redis 是否正常工作，如果它正常运行没有任何错误，那么你讲看到以下信息
```shell
● redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2020-04-04 21:43:41 CST; 18s ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 22633 ExecStopPost=/bin/run-parts --verbose /etc/redis/redis-server.post-down.d (code=exited, status=0/SUCCESS
  Process: 22630 ExecStop=/bin/kill -s TERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 22627 ExecStop=/bin/run-parts --verbose /etc/redis/redis-server.pre-down.d (code=exited, status=0/SUCCESS)
  Process: 22646 ExecStartPost=/bin/run-parts --verbose /etc/redis/redis-server.post-up.d (code=exited, status=0/SUCCESS)
  Process: 22642 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exited, status=0/SUCCESS)
  Process: 22640 ExecStartPre=/bin/run-parts --verbose /etc/redis/redis-server.pre-up.d (code=exited, status=0/SUCCESS)
 Main PID: 22645 (redis-server)
    Tasks: 3 (limit: 4915)
   CGroup: /system.slice/redis-server.service
           └─22645 /usr/bin/redis-server 127.0.0.1:6379

Apr 04 21:43:41 iZbp1f8pn0jp3j4ogroxpeZ systemd[1]: Stopped Advanced key-value store.
Apr 04 21:43:41 iZbp1f8pn0jp3j4ogroxpeZ systemd[1]: Starting Advanced key-value store...
Apr 04 21:43:41 iZbp1f8pn0jp3j4ogroxpeZ run-parts[22640]: run-parts: executing /etc/redis/redis-server.pre-up.d/00_exampl
Apr 04 21:43:41 iZbp1f8pn0jp3j4ogroxpeZ run-parts[22646]: run-parts: executing /etc/redis/redis-server.post-up.d/00_examp
Apr 04 21:43:41 iZbp1f8pn0jp3j4ogroxpeZ systemd[1]: Started Advanced key-value store.
```
通过修改 ```/etc/redis/redis.conf``` 文件中的 ```requirepass``` 参数来设置安全秘钥，避免 redis 被非正常访问。秘钥的长度越长，随机性越大保护效果越过，我们可以选择使用 ```openssl``` 来生成随机密码，而不是自己来设定一个密码。
```shell
openssl rand 60 | openssl base64 -A
```

## 在 Flask 中添加 Redis 的支持
在 Flask 中使用 Redis 可以直接使用 ```flask-redis``` 支持包，它是对 ```redis.py``` 的扩展，使用起来非常方便。使用以下命令即可安装该支持包：
```shell
pip install flask-redis
```

```flask-redis``` 的配置非常方便，只需要在配置文件中增加 ```REDIS_URL``` 的配置即可。
```
REDIS_URL = "redis://:password@localhost:6379/0"
```
```flask-redis``` 初始化同样非常简单，只需要两行代码即可。
```
redis_client = FlaskRedis()
...
redis_cline.init_app(app)
```
> 建议将 Redis 对象的获取同与 Flask 对象的挂载代码分开，便于代码的模块化结构。

## 在 Flask 添加动态数据
首先创建使用 Redis 存储/获取动态数据的函数，代码如下：
```python
def mark_dyn_data(id, data):
    user_id = str(id).encode('utf-8')
    data = str(data).encode('utf-8')
    expires = int(time.time()) + 60
    data_key = "dyn_data/%s" % user_id
    p = redis_client.pipeline()
    p.set(data_key, data)
    p.expireat(data_key, expires)
    p.execute()

def get_dyn_data(id):
    user_id = str(id).encode('utf-8')
    data_key = "dyn_data/%s" % user_id
    data = redis_client.get(data_key)

    if data:
        return int(data)
    return None
```
在 Redis 中使用键值对来存储数据，在键中增加用户 Id 作为唯一表示，在读取时同样以用户 ID 作为标识来进行查询。在代码中设置超时时间为 60 秒，当动态数据超过 60 没有更新时，Redis 会自动清除该数据。

在这里我们通过管道 ```pipeline``` 来操作 ```redis``` 以减少客户端与 redis-server 的交互次数。

完整代码请参考：https://github.com/keinYe/erp-server
