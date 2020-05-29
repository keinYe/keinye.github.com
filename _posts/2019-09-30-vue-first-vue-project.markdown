---
layout: post
title:  "使用 Vue 建立第一个前端项目"
date:   2019-09-30 20:57:22 +0800
author: keinYe
categories: [ Vue ]
image: assets/images/14.jpg
beforetoc: "."
toc: true
tags: [python, flask, login, 用户管理, API]
---
> Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

以上是 Vue 官方对它的介绍，简单来说 Vue 是一个使用 JavaScript 开发的前端框架，用于快速的搭建前端开发工程，Vue 还提供和兼容很多的组件，便于开发人员扩展功能。如果你想开发一个 web 界面，使用 Vue 是个不错的选择。

Vue 提供了一个官方的的命令行工具，我们可以通过 npm 来进行安装。
```shell
npm install vue
```
Vue 命令行工具安装完成后，可以通过命令行来生成工程目录。
```
vue create vue-project
```
命令执行完成，会看到以下界面
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-09-299.15.54.png)

此时应选择使用 Manaually select features，因为默认选择是没有
vue-router 的。

![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-09-299.18.41.png)
选择

接下来选择 Router 和 Vuex。

一路 Enter 等待依赖包安装完成后，在命令行输入 npm run serve，然后在浏览器地址栏输入 http://localhost:8080/ ,即可看到以下界面。
![](https://lg-8wz4hass-1252833766.cos.ap-shanghai.myqcloud.com/pic/屏幕快照2019-09-308.40.21.png)

> Vue 官方并不推荐新手使用命令行工具，推荐使用<script></script>标签在 HTML 文件中引入 Vue，由于本人实在是太白的小白还是使用了命令工具。
