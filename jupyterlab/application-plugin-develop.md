---
nav:
  title: jupyter lab 应用插件开发
  order: 2
---

# jupyter lab 应用插件开发介绍

jupyterlab 的功能由应用插件组成，核心的应用插件包括主菜单系统、文件浏览器、notebook、控制台、文本编辑器等等；应用插件之间可以依赖以及提供服务等方式运转。

## 开发教程

### 一、搭建开发环境

---

#### 1.1 下载 miniconda ，并按照文档安装

下载地址：<https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html>

#### 1.2 创建并激活虚拟环境

```
conda create -n jupyterlab-ext --override-channels --strict-channel-priority -c conda-forge -c anaconda jupyterlab cookiecutter nodejs git
```

```
conda activate jupyterlab-ext
```

### 二、创建 jupyterlab 插件项目

---

#### 2.1 初始化下载项目

```
cookiecutter https://github.com/jupyterlab/extension-cookiecutter-ts
```

安装提示输入

```
author_name []: Your Name
python_name [myextension]: jupyterlab_oss
labextension_name [myextension]: jupyterlab_oss
project_short_description [A JupyterLab extension.]: Show a random NASA Astronomy Picture of the Day in a JupyterLab panel
has_server_extension [n]: n
has_binder [n]: y
repository [https://github.com/my_name/oss]: https://github.com/my_name/jupyterlab_oss
```

进入目录并列出文件

```
cd jupyterlab_oss
```

安装项目依赖以及安装插件到 jupyterlab 环境中

```
pip install -ve .
```

创建链接指向源目录

```
jupyter labextension develop --overwrite .
```

给插件添加 watch 监听便于开发调试

```
jlpm run watch
```

新建一个终端窗口运行命令启动 jupyterlab

```
conda activate jupyterlab-ext
jupyter lab
```

浏览器中打开 jupyterlab 项目地址，如打印以下内容则表示插件安装成功。

```
JupyterLab extension jupyterlab_oss is activated!
```

### 三、插件开发

---

开发一个 demo 插件主要实现以下功能：

- 左侧 sidebar 中添加一个图标
- 图标激活的左侧区域中添加搜索框和列表
- 搜索框对列表进行搜索

#### 3.1 目录介绍

插件项目中需要关注 src/index.ts 文件和 style/index.css，这两个文件里面的内容分别是插件的逻辑和插件的样式

#### 3.2 插件对象属性介绍

开发插件之前得了解插件对象的一些属性，我们以初始化的 index.ts 文件为例。

```typescript
import {
  JupyterFrontEnd,
  JupyterFrontEndPlugin,
} from '@jupyterlab/application';

/**
 * Initialization data for the jupyterlab_oss extension.
 */
const extension: JupyterFrontEndPlugin<void> = {
  id: 'jupyterlab_oss:plugin',
  autoStart: true,
  activate: (app: JupyterFrontEnd) => {
    console.log('JupyterLab extension jupyterlab_oss is activated!');
  },
};

export default extension;
```

JupyterFrontEndPlugin：JupyterFrontEnd 应用程序插件的类型。包含：

- id（必需）-插件 ID，在应用程序是中唯一标识
- autoStart（可选）-应用程序启动时是否自动激活插件。默认是 false
- requires（可选）-插件所需的服务类型。当插件被激活时，实例会按照顺序传递给 activate()
- optional（可选）-插件的可选服务的类型。此 token 对应于插件可以使用的服务，可选服务将在所有必需服务之后传递给 activate（）函数。 如果不能提供可选服务，解析后，将传递“ null”代替。
- provides（可选）-插件提供的服务类型。此 token 对应于插件导出的服务。激活插件后，将使用“activate（）”的返回值作为该类型的具体实例。
- activate（必需）-调用以激活插件的功能。

JupyterFrontEnd：Jupyter 前端应用对象。包含：

- commands-在应用程序中添加和执行命令的可扩展注册表
- commandLinker-将 DOM 节点与命令注册表连接，以便单击它们即可执行命令。
- docRegistry-包含应用程序能够读取和呈现的文档类型的可扩展注册表。
- restored-应用程序完成加载后执行的 promise
- serviceManager-与 Jupyter REST API 会话的 low-level manager
- shell-构成应用程序用户界面的通用的 Jupyter 前端 Shell 实例

#### 3.3 插件开发思路

要实现上述的 demo 插件需求思路如下：

- 新建 factory 插件，给 oss 插件提供组件
- 新建 oss 插件；创建 factory 插件提供的组件实例，并添加到左侧 sidebar 中；将插件的命令添加到注册表中

定义 commands 命令

```typescript
namespace CommandIDs {
  export const toggleOSS = 'oss:toggle-main';

  export const showOSS = 'oss:activate';

  export const hideOSS = 'oss:hide-main';
}
```

插件定义代码

备注：index.ts 插件文件可以导出插件对象，也可以导出插件对象数组

```typescript
const oss: JupyterFrontEndPlugin<void> = {
  id: 'jupyterlab_oss:oss',
  requires: [IOSSFactory, ILabShell, ILayoutRestorer, ITranslator],
  autoStart: true,
  activate: activateOSS,
};

const factory: JupyterFrontEndPlugin<IOSSFactory> = {
  id: 'jupyterlab_oss:factory',
  provides: IOSSFactory,
  requires: [ITranslator],
  optional: [ILabShell, IStateDB],
  activate: activateFactory,
};

const namespace = 'oss';

const plugins: JupyterFrontEndPlugin<any>[] = [factory, oss];

export default plugins;
```

插件激活方法这里就不贴出来了，可以直接下载源码看看；

#### 3.4 factory 插件激活过程

factory 插件提供了组件的实例，组件激活的过程如下

1. 定义 model，管理组件所需要的数据对象
2. 通过 ID 和 model 以及其他参数创建组件实例
3. 定义组件追踪对象，将新创建的组件放入到追踪对象中
4. factory 组件返回包括创建组件方法、默认组件实例、组件追踪等属性的对象

```typescript
async function activateFactory(
  app: JupyterFrontEnd,
  labShell: ILabShell | null,
  state: IStateDB | null,
): Promise<IOSSFactory> {
  const tracker = new WidgetTracker<OSS>({ namespace });
  const createOSS = (id: string, options: IOSSFactory.IOptions = {}) => {
    const model = new OSSModel({
      auto: options.auto ?? true,
      state:
        options.state === null
          ? undefined
          : options.state || state || undefined,
    });

    const restore = options.restore;
    const widget = new OSS({ id, model, restore });

    // Tracker the newly created oss
    void tracker.add(widget);

    return widget;
  };

  const defaultOSS = createOSS('oss', {
    auto: false,
    restore: false,
  });

  return { createOSS, defaultOSS, tracker };
}
```

demo 中包含了搜索框、列表两个组件，并且搜索框能够对列表进行搜索，上述代码中的 OSS 组件类中实现该需求。

#### 3.5 jupyterlab 组件介绍

jupyterlab 基础组件分了 Widget、Layout;

开发插件，一般会有入口的 widget，widget 中的 layout 属性，包括了插件的子组件以及相应的布局，在插件激活的代码中，会挂载到页面相应的区域。

3.5.1 Widget

widget 是基础的元素组件，定义了组件的生命周期以及一些消息定义发布器(onBeforeAttach、onAfterAttach 等等),后代组件主要介绍下 ReactWidget、VDomRenderer。

- ReactWidget 继承 widget, 组件定义了与 React 组件相关的方法；如 render、reactDom 以及重写了 widget 生命周期方法
- VDomRenderer 继承了 ReactWidget 组件, 是 Model 与 ReactWidget 的组合，通过 model 的变化去触发 ReactWidget 组件的 stateChanged 更新事件,当然也可以根据业务逻辑需要去触发 stateChanged 更新事件

  3.5.2 Layout

Layout 是基础的布局组件，定义了一些属性和方法，去通知其布局内的组件进行添加、删除、更新等等操作；后代组件包括了 PanelLayout、DockLayout 等等。

#### 3.6 插件组件开发思路

我们以需求中的列表组件为例。

1. 定义组件参数结构 IOSSListProps
2. 定义组件 Model；Model 中定义了组件渲染所需要的 data 属性，并且通过 set data 方法，数据一旦发生变动就会触发组件状态更新的事件(this.stateChanged.emit(void 0)，通知组件更新视图)
3. 创建 OSSList 组件继承 VDomRenderer，并且在构造方法中创建 model 实例，将传入的参数赋值给 OSSList 组件的 model 属性
4. OSSList 中定义 render 方法

代码如下：

```typescript
export interface IOSSListProps {
  data?: IOSSListProps.IData[];
}

export namespace IOSSListProps {
  export interface IData {
    value: string;
    key: string;
  }
}
const OSSLISTCOMPONENTS = (props: IOSSListProps) => {
  const str = props.data
    ? props.data.map((item: IOSSListProps.IData) => {
        return <li key={item.key}>{item.value}</li>;
      })
    : [];

  return <ul>{str}</ul>;
};

export class OSSList extends VDomRenderer<OSSList.Model> {
  constructor(opts: IOSSListProps) {
    super(new OSSList.Model(opts.data));
  }

  render() {
    return <OSSLISTCOMPONENTS data={this.model.data}></OSSLISTCOMPONENTS>;
  }
}

export namespace OSSList {
  export class Model extends VDomModel {
    constructor(data: IOSSListProps.IData[]) {
      super();
      this._data = data;
    }

    get data() {
      return this._data;
    }

    set data(data: IOSSListProps.IData[]) {
      this._data = data;
      //触发组件更新视图
      this.stateChanged.emit(void 0);
    }

    private _data: IOSSListProps.IData[];
  }
}
```

### 四、打包发布

---

- 将 registry 切到你要发布的源上

```shell
npm set registry http://registry.npmjs.org
```

- 已注册用户执行下面命令（需输入账号登录，用户名，邮箱，密码）。未注册用户先去注册：<https://www.npmjs.com/>

```shell
npm login
```

- 发布插件包

```shell
npm publish
```

### 五、使用发布插件

---

```shell
jupyter labextension install '已发布插件的名字'
```
