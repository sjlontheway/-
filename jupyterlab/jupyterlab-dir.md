---
nav:
  title: jupyter lab 目录结构
  order: 2
---

# jupyter lab 目录解析

以下是jupyter lab项目根目录下的一些主要文件夹概要介绍

### packages
jupyter lab 前端包，jupyter 是以menorepo(单仓库多npm包的形式)管理前端代码的，此目录是jupyter lab实际业务所在地方

### dev_mod
打包构建相关目录,webpack相关插件，配置文件在这个目录下面
入口文件为index.js文件，jupyter lab应用启动，加载插件，激活插件，运行应用，是在此文件中进行。

### builder
构建相关脚本，以及webpack插件等

### builderUtils
构建工具库

### design 
关于jupyter lab功能设计的动机，讨论

###  jupyterlab
服务端python相关代码

### docs
jupyterlab 文档相关代码，前端同学不必太过关注

### examples
examples目录下有许多jupyterlab 组件的使用例子，进入对应目录，运行build指令构建完成之后，使用 `python main.py` 运行example

