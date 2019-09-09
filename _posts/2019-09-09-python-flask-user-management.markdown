---
layout: post
title:  "为 Flask 添加用户登录"
date:   2019-09-09 21:37:22 +0800
categories: python
tags: [python, flask, login, 用户管理, API]
---
Flask 是什么？我想打开这篇文章的你应该不陌生，但是我还引用维基百科上的内容做个简短的介绍。

> Flask 是一个使用 Python 编写的轻量级 Web 应用框架。基于 Werkzeug WSGI 工具箱和 Jinja2 模板引擎。Flask 使用 BSD 授权。<br>
> Flask 被称为 “microframework”，因为它使用简单的核心，用 extension 增加其他功能。Flask 没有默认使用的数据库、窗体验证工具。然而，Flask 保留了扩增的弹性，可以用 Flask-extension 加入这些功能：ORM、窗体验证工具、文件上传、各种开放式身份验证技术。

简单来说 Flask 是一个使用 Python 语言的 Web 服务框架，但是 Flask 仅实现了部分功能，大多数功能通过扩展来实现，使用者可以用自己最熟悉的模块来实现自己的功能。

当然今天这篇文章不是来介绍 Flask 的，而是如何在 Flask 中增加用户管理「用户登录」的功能。Flask 是一个 Web 框架，在服务端需要实现的用户登录主要有两种方式，一个是通过网页登录，另一个是通过 API 登录。这里将带你实现这两种方式的用户登录。

## 网页中的用户登录实现
在 Flask 中网页的用户登录，主要通过 Flask-Login 扩展来完成，
通过 Flask-Login 可以实现以下功能：
- 存储会话中活动用户的 ID，并允许你随意登入登出。
- 让你限制已登入（或已登出）用户访问视图。
- 实现棘手的“记住我”功能。
- 保护用户会话免遭 Cookie 盗用。
- 随后可能会与 Flask-Principal 或其它认证扩展集成。

在使用 Flask-Login 之前需要用 pip 来安装它，对 Flask-Login 的使用主要分为以下几个步骤。

一、建立 LoginManager 的实例，并使用 Flask 初始化 Flask-Login。
```python
loginmanager = LoginManager()
loginmanager.init_app(app)
```
二、建立 user_loader 的回调函数，该函数通过 user_id 来获取 user 对象。
```python
    @login_manager.user_loader
    def load_user(user_id):
        """Loads the user. Required by the `login` extension."""
        user_instance = User.query.filter_by(id=user_id).first()
        if user_instance:
            return user_instance
        else:
            return None
```
三、建立用户类。用户类继承自 UserMixin，它提供了 is_authenticated、is_active、is_anonymous、get_id 等方法的默认实现。

四、建立用户登录页面，需要包含输入框及提交按钮。

五、建立登录视图，并在登录验证完成是调用 login_user 函数。
```python
class Login(MethodView):
    def get(self):
        return render_template('user/login.html')

    def post(self):
        username = request.form['username']
        password = request.form['password']
        remember = True if request.form.get('remember') else False
        logger.info(remember)
        user = User.query.filter_by(name=username).first()
        if user and user.check_password(password):
            login_user(user, remember=remember)
            next = request.args.get('next')
            return redirect(next or url_for('brand.bar'))
        return render_template('user/login.html')
```

程序实现到现在，用户登录的整个过程已经完成，可以通过用户名和密码来实现用户的验证，但是你会发现所有的 url 你还是可以在没有登录的状态下访问，那么如何使需要登录的 url 处于保护状态呢？答案是使用 login_required 标签。
```python
class BrandBar(MethodView):
    decorators = [login_required]

    def get(self):
        brands = [name[0] for name in db.session.query(Brands.name).all()]
        # brands = db.session.query(Brands).all()
        return render_template('brand_bar.html', brands=brands)
```

当你访问一个需要登录才能访问的视图时，希望能够自动跳转到登录页面，此时还需要设置一个默认的登录视图，设置方法如下：
```python
login_manager.login_view = 'user.login'
```

现在整个登录过程才算完整，当你访问 BrandBar 视图时，未登录时会自动跳转到登录页面，完整登录后会自动跳转到 BrandBar 视图。


## API 中的用户登录实现
REST API 是通过 API 来访问服务端数据，服务端返回的数据通常是 JSON 格式，API 的用户登录实现我们通过 flask_httpauth 来完成。通过 flask_httpauth 可以通过 token 来登录，而不需要每次都在 http 请求中携带用户名和密码。

在使用 flask_httpauth 前需要先安装「采用统一的 pip 来安装 python 模块」和初始化 flask_httpauth。
```python
from flask_httpauth import HTTPBasicAuth

auth = HTTPBasicAuth()
```

设置 verify_password 回调函数。
```python
@auth.verify_password
def verify_password(username_or_token, password):
    # first try to authenticate by token
    user = User.verify_auth_token(username_or_token, current_app)
    logger.info('username:{} password:{}'.format(username_or_token, password))
    if not user:
        # try to authenticate with username/password
        user = User.query.filter_by(name = username_or_token).first()
        if not user or not user.check_password(password):
            return False
    g.current_user = user
    return True
```
> verify_password 回调函数同时接收用户名/密码、token 两种方式来完成登录。在用户类中需要实现通过密码和 token 进行验证的方法。

```python
class User(db.Model, UserMixin, CRUDMixin):

    ....

    def check_password(self, password):
        """Check passwords. If passwords match it returns true, else false."""

        if self.password is None:
            return False
        return check_password_hash(self.password, password)

    def generate_auth_token(self, app, expiration = 600):
        s = Serializer(app.config['SECRET_KEY'], expires_in = expiration)
        return s.dumps({ 'id': self.id })

    @staticmethod
    def verify_auth_token(token, app):
        s = Serializer(app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        # except SignatureExpired:
        #     return None # valid token, but expired
        except BadSignature:
            return None # invalid token
        user = User.query.get(data['id'])
        return user
```
是在登录完成后通过 API 来获取 token ，以后访问 API 可以直接携带 token 无需使用用户名和密码进行登录。

通过以下命令实现通过用户名和密码来获取认证 token
```
curl -u test:test -i -X GET http://127.0.0.1:5000/api/v0.1/user/token
```
获取 token 后，即可通过 token 来访问
```
curl -X GET -H "Authorization: Bearer secret-token-1" http://127.0.0.1:5000/api/v0.1/user/token
```
> 需要经过认证才能访问的 API 请至于 auth.login_required 的保护之中。
## 后记
本次实现网页端和 API 端简单的用户认证功能，还需许多功能需要进一步发现，比如 flask_httpauth 的其他认证方式，网页端和 API 之间的认证打通，这些功能留待以后进一步学习吧！
