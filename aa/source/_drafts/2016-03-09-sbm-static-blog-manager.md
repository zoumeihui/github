title: 跨平台静态博客管理工具设计
layout: post
date: 2016-03-09 15:39:45
categories: design
tags: [visualization, node.js, electron]
---

## 沙碧马

软件名称: SBM (Static Blog Manager)

跨平台的Github静态博客文章发布桌面程序.

![Screenshot](http://w3log.qiniudn.com/sbm.png)

<!-- more -->

## 功能介绍

### 系统功能

* Github账号登录,选择一个Repo作为工作目录
* 注销登录,和切换用户登录
* 切换Repo工作目录
* 版本更新检测

### 文章发布管理

* 设置文章目录
* 目录和文章管理
* Markdown文章编辑,实时本地存储
* 本地内容与服务器差异比较

## 设计思路

使用JS开发跨平台应用,框架选用[Electron](http://electron.atom.io/).

### 参考资料

1. Electron框架: <http://electron.atom.io/>
2. Github接口文档: <https://developer.github.com/v3/>
3. MD渲染 Marked: <https://github.com/chjj/marked>
4. MD编辑器 Pen: <https://github.com/sofish/pen>

## 代码贡献

欢迎贡献代码: <https://github.com/willin/sbm>