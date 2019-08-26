---
layout: post
title:  "外行学 python 爬虫 第七篇 开启多线程"
date:   2019-06-18 21:37:22 +0800
categories: python
tags: [python, 外行, 爬虫]
---
经过上一篇文章[外行学 Python 爬虫 第六篇 动态翻页](https://mp.weixin.qq.com/s/xBEKaYyAGE_2rS8-b27uKw)我们实现了网页的动态的分页，此时我们可以爬取立创商城所有的原件信息了，经过几十个小时的不懈努力，一共获取了 16万+ 条数据，但是软件的效率实在是有点低了，看了下获取 10 万条数据的时间超过了 56 个小时，平均每分钟才获取 30 条数据。

![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-06-15下午10.14.28.png)
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-06-15上午6.58.27.png)

> 注：软件运行的环境是搬瓦工的虚拟主机，CPU: 2x Intel Xeon , RAM: 1024 MB

软件的运行效率不高，那么时间都花费在什么上面了，爬虫软件本身并不是计算密集型软件，时间大多数花费在与远程主机的通信上了，要想提高软件的运行效率，就要减少等待时间，此时你想到了什么？没错就是多线程，在非计算密集型应用中，使用多线程可以最大程度的节省资源同时提高软件的效率，关于线程的基本应用可以参考前面的文章 [python 之进程与线程](https://mp.weixin.qq.com/s/KIuRVGAHfNxuz3T8FV5iuw)。

## 针对多线程的修改
使用多线程后，每个线程执行的应该是不同的任务，如果是相同的任务那就是两个程序而不能说是多线程了。每个线程执行不同的任务「即爬取不同的网页」，需要线程间共享数据「在本程序中需要共享待爬队列、已获取 url 的布隆滤波器等」。因此我们需要多当前的软件进行修改，以使待爬队列和布隆滤波器可以在多个线程之间共享数据。

要想在多线程之间共享待爬队列和布隆滤波器，需要将其从当前的实例属性修改为类属性，以使其可以通过类在多个线程中访问该属性。关于类属性和实例属性可以参考 [Python 类和实例](https://mp.weixin.qq.com/s/asc_QgNoTI1w69LZjbVoBA) 这篇文章。

将待爬队列和布隆滤波器设置为类属性的代码如下：
```python
class Crawler:
    url_queue = Queue()
    bloomfilter = ScalableBloomFilter()
    ...
```
在使用的过程中通过类名来访问类属性的值，示例代码如下：
```python
    def __init__(self, url_count = 1000, url = None):
        if (Crawler.max_url_count < url_count):
            Crawler.max_url_count = url_count

        Crawler.url_queue.put(url)
```

在多线程中，当前的类属性有多个线程共享，任何一个类属性都有可能被任何线程修改，因此线程之间共享数据最大的危险在于多个线程同时修改一个数据，把数据给修改乱了。由于 Queue 是一个适用于多线程编程的先进先出的数据结构，可以在生产者和消费者线程之间安全的传递消息或数据，因此我们无需对队列进行操作，但是布隆滤波器是非线程安全的数据，此时我们就需要在修改布隆滤波器的地方加上线程锁，以保证在同一时刻只有一个线程能够修改布隆滤波器的数据，代码如下：
```python
    def url_in_bloomfilter(self, url):
        if url in Crawler.bloomfilter:
            return True
        return False
    def url_add_bloomfilter(self, url):
        Crawler.lock.acquire()
        Crawler.bloomfilter.add(url)
        Crawler.lock.release()
```
在所有需要判断 url 是否已经爬取过的地方调用 url_in_bloomfilter，当需要向布隆滤波器中添加 url 时调用 url_add_bloomfilter 方法，保证布隆滤波器的数据不会被错误修改。

对爬虫类 Crawler 修改完成后，就是真正启动多线程的时候，在 main.py 文件中将代码修改为如下内容：
```python
def main():
    with open('database.conf','r') as confFile:
        confStr = confFile.read()
    conf = json.JSONDecoder().decode(confStr)
    db.init_url(url=conf['mariadb_url'])

    crawler1 = Crawler(1000, url='https://www.szlcsc.com/catalog.html')
    crawler2 = Crawler(1000, url='https://www.szlcsc.com/catalog.html')
    thread_one = threading.Thread(target=crawler1.run)
    thread_two = threading.Thread(target=crawler2.run)
    thread_one.start()
    thread_two.start()
    thread_one.join()
    thread_two.join()
```
以上代码中首先建立了对数据库的连接，然后创建了两个 Crawler 类的的实例，最后创建了两个线程实例，并启动线程。


## 修改后的执行结果
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-06-16下午10.19.36.png)
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-06-17上午6.45.38.png)
本次软件开启了两个线程同时运行，同样获取 10 万条数据，一共花费了 29 个小时，平均每分钟获取了 57.5 条数据，相比单线程效率提高了 191.7%，总体来说效率提高还是非常明显的。

![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-06-18下午2.55.53.png)
最终在花费 50 小时 30 分钟，从立创商城上获取十六万五千条数据后，程序执行完成。
> 从立创商城商品目录页面可知立创商城上共计有十六万七千个元件。程序执行完成后共计获取十六万五千条数据，可以说完成了预期设计目标。
