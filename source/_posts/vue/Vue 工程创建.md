---
type: blog
title: Vue 工程创建
date: 2021-03-01
categories: 技术
tags: vue
cover: false
meta:
  date: false
---





<!-- more -->

## 一、初始化项目

### 1.1 Vue UI 创建

- 命令行输入，会跳出一个浏览器页面：

```bash
vue ui
```

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210301172457715.png" alt="image-20210301172457715" style="zoom: 25%;" />

- 选择一个存放路径
- 输入项目名称
- 进入「创建新项目」界面
  - 如果之前有保存配置，这里可以选择之前的
  - 没有保存过，就选手动

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210301173027563.png" alt="image-20210301173027563" style="zoom:25%;" />

- 进入「功能」界面
  - 添加一些常用的功能
  - 一般会需要：Babel、Router、Vuex、Linter/Formatter、使用配置文件
- 进入「配置」页面
  - Linter/formatter 选择 standard

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210301173626015.png" alt="image-20210301173626015" style="zoom:25%;" />

- 最后可以把配置保存

- 生成项目大概需要5-10min



## 二、安装插件 & 依赖

### 2.1 插件

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210301174646908.png" alt="image-20210301174646908" style="zoom:33%;" />

**推荐安装的插件：**

- vue-cli-plugin-element
  - 想要生成的包小，就用「import on demand」；如果想开发省事，就「Fully import」

### 2.2 依赖

<img src="Vue 工程创建.assets/image-20210301175324844.png" alt="image-20210301175324844" style="zoom: 33%;" />

- 运行依赖
  - axios
- 开发依赖
  - less-loader
  - less



## 三、生产优化

- 移除 console 输出

- 查看打包报告

能够看到各种环境的占比；

或者使用命令行

```bash
vue-cli-service build --report
```



<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20210310101101647.png" alt="image-20210310101101647" style="zoom: 25%;" />

- 分开 prod 和 dev 的配置文件
- 使用 CDN 加载

减小包的大小

- 路由懒加载

[基于Vue、ElementUI的换肤解决方案](https://neveryu.github.io/2019/07/01/vue-element-change-theme/)

