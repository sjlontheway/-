---
nav:
  title: jupyter lab 启动过程
  order: 3
---

### Jupyter Lab 实例继承关系

JupyterLab实例继承关系有三层，``` JupyterLab extends | JupyterFrontEnd extends | Application ```

- @lumino/Application Application类
   第三方插件可通过继承Application类来快速构建自己的系统，它提供插件注册、激活功能、全局事件处理功能
  、命令注册CommandRegistry实例来注册命令、绑定命令快捷键、拦截处理全局事件

- @jupyterlab/Application JupyterFrontEnd类
  Jupyter 针对前端的通用抽象类，继承Application类，在构造函数中创建命令链接器，文档注册器，与后端交互的
  ServiceManager 类

- @Jupyterlab/Application JupyterLab类
  JupyterLab 应用类，构造函数中主要是初始化应用配置信息，注册默认插件以及注册默认文档格式，监听窗体模式
  来动态显示窗口

### Jupyter Lab 插件注册流程

![avatar](/debugger_protocol_diagram.png)

  ```flow

   st=>start: Start
   i=>inputoutput: 输入年份n
   cond1=>condition: n能否被4整除？
   cond2=>condition: n能否被100整除？
   cond3=>condition: n能否被400整除？
   o1=>inputoutput: 输出非闰年
   o2=>inputoutput: 输出非闰年
   o3=>inputoutput: 输出闰年
   o4=>inputoutput: 输出闰年
   e=>end
   st->i->cond1
   cond1(no)->o1->e
   cond1(yes)->cond2
   cond2(no)->o3->e
   cond2(yes)->cond3
   cond3(yes)->o2->e
   cond3(no)->o4->e

   ```
