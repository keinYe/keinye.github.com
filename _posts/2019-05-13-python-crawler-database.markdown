---
layout: post
title:  "外行学 python 爬虫 第五篇 数据存储"
date:   2019-05-13 21:37:22 +0800
categories: python
tags: [python, 外行, 爬虫]
---
前面一至四篇我们学习了如何使用 python 来获取网页并将网页中的有效数据解析出来，当获取到有效数据以后，不可能将数据放在内存中，一旦系统出现问题辛辛苦苦获取的数据都付诸东流了，此时需要考虑数据持久化的事情，数据持久化我们有两种选择一是将数据保存在文件中「比如 txt 文件或 execl 文件」，另一种是将数据保存在数据库中。

对于将数据保存到文件中前面已经写过相应的文件有兴趣的话可以看 [保存数据到文件](https://mp.weixin.qq.com/s/CvUh_XP_P2b8n1oUfn-YSQ) 这篇文件，今天我们主要来看下如何将获取到的有效数据保存在数据库中。

将数据保存到数据库首先需要使用 python 连接到数据，并依据数据的类型创建数据类，[Python 数据库操作 SQLAlchemy](https://mp.weixin.qq.com/s/4yMn3CMYzH_R8vJB11ByUg) 这篇文章详细介绍了如何在 python 中使用 SQLAlchemy 库连接数据并创建数据表，[SQLAlchemy 定义关系](https://mp.weixin.qq.com/s/YxMa2qSkUyn-UwyWcjI2FQ) 这篇文件详细介绍了如何使用 SQLAlchemy 来建立各个数据表之间的关系。

在前面的文章中使用 BeautifulSoup 解析出了网站立创商城中的电子元件的基本信息「厂家、型号、名称等等」以及相应的价格信息。因为电子元件的基本信息时固定不变，而价格信息却是浮动的，如果我们想要建立该电子元件的价格波动情况，就需要有它在不同时期的价格，此时如果将基本信息和价格信息使用同一张表来实现的话，是无法完成了此功能的。需要将基本信息和价格信息分开，一个电子元件在不同的时期可以对应多个不同的价格，这是一个一对多的关系。

具体的数据表信息如下：
```python
class Materials(Base, CRUDMixin):
    __tablename__ = 'materials'

    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100), nullable=False)
    category = Column(String(100), nullable=False)
    brand = Column(String(100), nullable=False)
    model = Column(String(100), nullable=False)
    number = Column(String(100), unique=True, nullable=False)
    package = Column(String(50), nullable=False)
    # price
    price = relationship('Price', backref='materials')

class Price(Base, CRUDMixin):
    __tablename__ = 'price'

    id = Column(Integer, primary_key=True, autoincrement=True)
    date = Column(DateTime, default=datetime.datetime.utcnow)
    price = Column(Text)
    materials_id = Column(Integer, ForeignKey('materials.id'))

```

在数据爬取的过程中，有可能长时间获取到的是无效的数据，此时会产生一段没有对数据库进行操作的时间，可能造成数据库链接的断开，需要在 SQLAlchemy 的初始化中设置自动重连，避免出现无法存储数据的情况。设置代码如下：
```python
    def init_url(self, url):
        self.engine = create_engine(
            url,
            pool_size=100,
            pool_recycle=5,
            pool_timeout=30,
            pool_pre_ping=True,
            max_overflow=0)
        self.db_session = sessionmaker(bind=self.engine)
        self.session = self.db_session()
        Base.metadata.create_all(self.engine)
```
使用一个单核 1G 内存的服务器，用 40 多个小时的时间一共获取了一共获取四万两千多条的数据
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-05-1310.33.05.png)
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-05-0910.14.16.png)
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-05-0910.14.57.png)

完整[代码地址](https://github.com/keinYe/pycrawler)

更多内容关注我的微信公众号「keinYe」。
