---
layout: post
title:  "外行学 python 爬虫 第十一篇 数据可视化"
date:   2019-08-23 21:37:22 +0800
categories: python
tags: [python, 外行, 爬虫]
---
在 [外行学 Python 爬虫 第九篇 读取数据库中的数据](https://mp.weixin.qq.com/s/ZPn3qXBVURtpKXGvtAU2nA) 中完成了使用 API 从数据库中读取所需要的数据，但是返回的是 JSON 格式，看到的是一串的字符串数据不是很好理解，这篇将介绍如何将数据进行可视化。

数据可视化选用 pyecharts 来完成，通过将 pyecharts 集成到 Flask 中完成数据从数据库到网页可视化显示的过程。最终完成后通过选择框选中相应的生产商后，即可查看在立创商城上该厂商所生产各种元件的数量，如下图所示：
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-08-23下午8.07.06.png)

> pyecharts 是利用 Echarts 生成图标的 Python 类库，是一款强大的数据可视化工具。更多信息请查阅 [pyecharts 官网](https://pyecharts.org/#/)。

## 准备工作

### pyecharts 安装
pyecharts 可是使用 pip 进行安装
```shell
pip install pyecharts
```
也可以使用源码进行安装
```shell
$ git clone https://github.com/pyecharts/pyecharts.git
$ cd pyecharts
$ pip install -r requirements.txt
$ python setup.py install
```
> 推荐使用 pip 进行安装，特别是对于初学者来说。

### 集成到 Flask 中
需要将 pyecharts 中的模板拷贝到 Flask 目录下的 templates 目录中，模板文件位于 ```pyecharts/pyecharts/render/templates/```目录中。

实际上此时即可在 Flask 中使用 pyecharts 了，但是根据 pyecharts 文档中的介绍，在实际使用过程中遇到了以下错误
```shell
jinja2.exceptions.TemplateNotFound: simple_chart.html
```
该错误是由以下配置引起的
```shell
CurrentConfig.GLOBAL_ENV = Environment(loader=FileSystemLoader("./templates"))
```
将该配置从代码中删除，重新运行程序即可看到完整的图标信息。

## 爬虫数据可视化
在这里将完成从数据库中读取各生产商所生产各类元件的数据，通过 Echarts 进行可视化的操作。为了实现能够通过选择生产商实时更新图表数据，最终使用前后端分离的方法实现数据显示。

首先，先看下 HTML 文件
```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <script src="https://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"></script>
    <script type="text/javascript" src="https://assets.pyecharts.org/assets/echarts.min.js"></script>
</head>
<body>
    <select id="brand_name" onchange="showBrandBar()">
      {% for brand in brands %}
      <option value="{{brand}}">{{brand}}</option>
      {% endfor %}
    </select>
    <div id="bar" style="width:1000px; height:600px;"></div>
    <script>
        $(
          showBrandBar()
        )

        function showBrandBar() {
          var selId = document.getElementById("brand_name");
          var value = selId.options[selId.selectedIndex].value;

          console.log(value)
          postBrandBar(value)
        }

        function postBrandBar(brand_name) {
          var chart = echarts.init(document.getElementById('bar'), 'white', {renderer: 'canvas'});
          $.ajax({
              type: "POST",
              url: "http://127.0.0.1:5000/brand/bar",
              dataType: 'json',
              data:JSON.stringify({
                name: brand_name
              }),
              success: function (result) {
                  chart.setOption(result);
              }
          });
        }
    </script>
</body>
</html>
```
采用 ajax 来响应 select 标签的改变事件，通过 ajax 向服务端提交当前选中的生产商，同时从服务器获取该厂商的信息。可以看到在 javascrpit 脚本中默认执行了一次 showBrandBar() 函数，是为了在第一次加载页面时能够正常显示图标，否则第一次加载页面时图表位置将是空白。

在 Flask 的后端需要实现一个 get 方法和一个 post 方法。get 方法用来获取所有的生产商名称，同时向浏览器发送 html 页面；post 方法用来相应 html 页面中的 ajax 请求，发送该生产商所提供的各类元件的数量。

get 方法的代码如下
```python
    def get(self):
        brands = [name[0] for name in db.session.query(Brands.name).all()]
        return render_template('brand_bar.html', brands=brands)
```
post 方法的代码如下
```python
    def post(self):
        jsons = request.get_json(force=True)
        brand_name = jsons['name']
        brand_id = db.session.query(Brands.id).filter(Brands.name == brand_name).one()[0]

        catalog_ids = db.session.query(Catalog.id, Catalog.name).filter(Catalog.parent_id==None).all()

        catalog_name = []
        catalog_count = []
        for cata_id, name in catalog_ids:
            catalog_name.append(name)
            count = db.session.query(Materials).join(Catalog).filter(
                    Materials.brand_id == brand_id).filter(
                    Catalog.parent_id == cata_id).count()
            catalog_count.append(count)

        c = (
            Bar()
            .add_xaxis(catalog_name)
            .add_yaxis('数量', catalog_count)
            .set_global_opts(
                xaxis_opts=opts.AxisOpts(axislabel_opts=opts.LabelOpts(rotate=-90)),
                title_opts=opts.TitleOpts(title=brand_name, subtitle="物料数量"),
            )
        )
        return c.dump_options_with_quotes()
```
在 post 方法中主要做的是数据的查询，通过生产商名称来查询出该生产商在数据中的 id，从而获取其所提供的所有元件，然后按照 Catalog 中的分类获取其各个分类中的元件数据。将相应的数据填入 pyecharts 的 Bar 对象中回传给 ajax 请求。

至此，执行程序在浏览器中即可看到在文章开头所看到的页面，选择不同的生产商图标将实时更新到该生产商的信息。
