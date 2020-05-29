---
layout: post
title:  "使用 uWSGI + Nginx 部署 Flask 应用"
date:   2019-12-11 20:57:22 +0800
author: keinYe
categories: [ Flask ]
image: assets/images/1.jpg
beforetoc: "."
toc: true
tags: [python, flask, uWSIG, Nginx]
---

在这篇文章之前，所有的应用都是在命令行使用 Python 直接运行的，但是这种方式只适合在开发过程中使用，并不适合在生产环境中使用，在生产环境中可以使用 uWSGI + Nginx 来部署程序。

>uWSGI 是一个软件应用程序，“旨在开发用于构建托管服务的完整堆栈”。它以 Web 服务器网关接口的名称命名，这是该项目支持的第一个插件。  uWSGI 通常用于与诸如 Cherokee 和 Nginx 之类的 Web 服务器一起为 Python  Web 应用程序提供服务，后者直接支持 uWSGI 的本机 uwsgi 协议。

>Nginx（发音同 engine x ）是异步框架的网页服务器，也可以用作反向代理、负载平衡器和 HTTP 缓存。Nginx 是免费的开源软件，根据类 BSD 许可证的条款发布。一大部分 Web 服务器使用 Nginx ，通常作为负载均衡器。

以上是维基百科中对 uWSGI 和 Nginx 的解释。

Flask 应用本质上是一个 WSGI 应用，在官方文档中推荐使用 Gunicorn、uWSGI、Gevent、Twisted Web 等 WSGI 服务器来部署 Flask 应用，Gunicorn 据说配置很简单，可惜一直没有成功过，这里还是使用 uWSGI + Nginx 来部署。

## 安装
uWSGI 可以直接使用 pip 来安装
```shell
pip install uwsgi
```
> 这里需要注意的是，如果程序运行在 Python3，uwsgi 需要使用 pip3 来进行安装，否则会出现各种意外。

Nginx 可以使用系统中自带的软件包管理工具进行安装，本人使用的是  Debian 系统系统，以下是安装命令。
```shell
apt-get update
apt-get install nginx
```
>安装过程可能需要 root 权限，请加上 sudo。

## 配置
首先，你需要一个 Flask 程序运行的入口文件，形式大致如下：
```python
# -*- coding:utf-8 -*-

from server import create_app

app = create_app()

if __name__ == '__main__':
    app.run()
```
在该文件中你需要暴露出 Flask 的对象，以提供给 uWSGI 使用。

其次，你需要完成名为 uwsgi.ini 的 uWSGI 的配置文件，文件内容大致如下：
```
[uwsgi]
socket          = 127.0.0.1:5000
chdir           = /face
module          = main:app
processes       = 2
threads         = 2
master          = true
daemonize       = /logs/uwsgi.log
pidfile         = uwsgi.pid
virtualenv      = /face/.venv
```
文件中各参数含义如下：
- **socket**: 设定 Flask 的地址和端口号。
- **chdir**: 设定 Flask 应用的根目录。
- **module**: 设定应用的入口文件及 Flask 对象。
- **processes**: 设定应用进程的数量。
- **threads**: 设定每个进程的线程数量。
- **master**: 设定是否启动主线程。
- **daemonize**: 设定日志的打印文件。
- **pidfile**: 设定主进程 pid 的写入文件。
- **virtualenv**: 设定虚拟环境的路径。

在 uwsgi.ini 文件中要特别主要 socket 参数一定要与 Flask 中设置的相同，Flask 默认的地址和端口号是 127.0.0.1:5000，如果你修改了默认值请记得修改这里。

最后，我们还需要配置 Nginx 反向代理，否则无法在外网进行访问。Nginx 的配置内容如下：
```
	server {
		# 监听端口
		listen 80;
		# 监听ip 换成服务器公网IP
		server_name ***.***.***.***;
		#动态请求
		location / {
			include uwsgi_params;
			uwsgi_pass 127.0.0.1:5000;
		}
		#静态请求
		location /static {
			alias /root/face/server/static/;
		}
	}
```
将以上内容添加到 ```/etc/nginx/nginx.conf``` 文件中，以上内容配置了 nginx 的监听端口以及公网 IP 地址，这里足以 uwsgi_pass 参数的值一定要保持与 uwsgi.ini 文件中一致。

在静态请求的配置中，一定要注意静态文件目录的用户权限，一般情况下 nginx.conf 文件首行会是 nginx 的用户组，如果该用户组无法访问你的静态文件目录，就会一直出现 502 错误，如果你有静态文件访问需求，请确认一下。

## 启动
启动分 uwsgi 的启动和 nginx 的启动。uswgi 的启动可使用命令
```shell
uwsgi --ini uwsgi.ini
```
如果你已经启动过 uwsgi 服务，先使用以下命令停止 uwsgi 在进行启动。
```
uwsgi --stop uwsgi.pid
```
或使用以下命令对 uwsgi 进行重启
```
uwsgi --reload uwsgi.pid
```

> 如果你使用 python 虚拟环境，尽可能在虚拟环境下启动 uwsgi。

Nginx 的启动使用以下命令
```shell
/etc/init.d/nginx start
```
Nginx 的停止使用以下命令
```shell
/etc/init.d/nginx stop
```
或使用以下命令重启 Nginx
```shell
/etc/init.d/nginx restart
```

当你正常启动 uWSGI 和 Nginx 以后，你就可以在浏览器中通过你服务器的 ip 地址来访问你自己的 Flask 应用了。
