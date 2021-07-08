---
nav:
  title: jupyter lab 插件介绍
  order: 2
---
# jupyter lab 插件简介


### 简介
Jupyterlab 应用是由一个核心应用 JupyterLab 对象以及一系列插件组成。

### 按构建方式区分插件类型
  
- 从发布类型来看分两种，一种是以源码的形式发布在npm，安装的时候需要重新构建按jupyterlab系统;

- 另外一种是以预构建扩展，就是将前端代码打包之后以python 包的形式发布，安装时，实质上是将bundle包复制至一个制定目录，

  注意：当同名包存在时，优先预购建安装的包。

### 按功能区分插件类型

- 应用插件
  如文件浏览器、控制台等
  一个应用插件包含如下元素：
``` typescript
const plugin: JupyterFrontEndPlugin<MyToken> = {
  id: 'my-extension:plugin',
  autoStart: true,
  requires: [ILabShell, ITranslator],
  optional: [ICommandPalette],
  provides: MyToken,
  activate: activateFunction
 };

```

- 数据渲染插件
  pdf 渲染插件、vega数据渲染插件等

- 主题插件
  ThemeLight主题插件等

### plugin 设置


### plugin 重写一个已存在的插件



### 重复依赖