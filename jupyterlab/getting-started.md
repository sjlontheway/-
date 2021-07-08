---
nav:
  title: jupyter lab 安装
  order: 1
---

# jupyter 安装环境准备

### 安装python3.x环境
安装notebook

``` shell

pip install notebook

```

### 安装nodejs 12+
jupyter lab 源码开发环境是nodejs 12+,engines 在dev_mode/package.json中限定

### 下载源码编译运行
``` shell
# 下载源码
git clone https://github.com/jupyterlab/jupyterlab.git
# 安装python 依赖 
cd jupyterlab && pip install -e .
# 安装npm 依赖
jlpm install
# 构建前端开发环境
jlpm run build:dev
# 启动本地开发环境，--watch 是实时编译更新ts代码帮助我们本地调试 
jupyter lab --dev-mod --wacth
```

执行完以上目录之后我们就可以运行jupyter lab服务体验jupyter lab的功能了
