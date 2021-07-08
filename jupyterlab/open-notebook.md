---
nav:
  title: Notebook文件打开过程
  order: 2
---

# Notebook 文件打开过程

## Notebook 文件打开之前的预备工作

系统在启动的时候，安装了 Notebook 插件，插件安装的内容具体如下：

1. factory; 组件单元格工厂提供者
2. tracker; 组件跟踪器提供者
3. tools; 工具栏插件
4. commandEditItem; 底部状态栏中 Mode 类型的切换插件
5. notebookTrustItem; 添加状态条目到右侧状态栏的插件
6. widgetFactoryPlugin; 组件工厂方法提供者
7. logNotebookOutput; console 日志打印插件

### factory 插件

根据编辑器服务插件，生成 Notebook 的内容工厂实例，提供给其他插件使用。

```typescript
//解读：编辑器的工厂服务提供了两个方法，一个是创建行内编辑器，一个是创建文档编辑器。目前的Notebook是行内编辑器。

//插件激活方法 packages/notebook-extension/src/index.ts 277行
const factory: JupyterFrontEndPlugin<NotebookPanel.IContentFactory> = {
  id: '@jupyterlab/notebook-extension:factory',
  provides: NotebookPanel.IContentFactory,
  requires: [IEditorServices],
  autoStart: true,
  activate: (app: JupyterFrontEnd, editorServices: IEditorServices) => {
    const editorFactory = editorServices.factoryService.newInlineEditor;
    return new NotebookPanel.ContentFactory({ editorFactory });
  },
};

//packages/codemirror/src/factory.ts 55行
newInlineEditor = (options: CodeEditor.IOptions) => {
  options.host.dataset.type = 'inline';
  return new CodeMirrorEditor({
    ...options,
    config: { ...this.inlineCodeMirrorConfig, ...(options.config || {}) },
    translator: this.translator,
  });
};
```

### widgetFactoryPlugin

依赖插件: factory 插件、编辑器服务、渲染媒体注册器、session 上下文窗口、国际化
提供： Notebook 组件工厂创建方法

```typescript
//插件激活方法 packages/notebook-extension/src/index.ts 530行
//解读：创建Notebook组件工厂实例，然后添加到文档注册器中
function activateWidgetFactory(
  app: JupyterFrontEnd,
  contentFactory: NotebookPanel.IContentFactory,
  editorServices: IEditorServices,
  rendermime: IRenderMimeRegistry,
  sessionContextDialogs: ISessionContextDialogs,
  translator: ITranslator,
): NotebookWidgetFactory.IFactory {
  const factory = new NotebookWidgetFactory({
    name: FACTORY,
    fileTypes: ['notebook'],
    modelName: 'notebook',
    defaultFor: ['notebook'],
    preferKernel: true,
    canStartKernel: true,
    rendermime: rendermime,
    contentFactory,
    editorConfig: StaticNotebook.defaultEditorConfig,
    notebookConfig: StaticNotebook.defaultNotebookConfig,
    mimeTypeService: editorServices.mimeTypeService,
    sessionDialogs: sessionContextDialogs,
    translator: translator,
  });
  app.docRegistry.addWidgetFactory(factory);
  return factory;
}
```

## Jupyterlab 打开 Notebook 文件的过程

1.  执行 packages/filebrowser-extension/src/index.ts 654 行

```typescript
//解读：执行open命令，如果类型是文件夹，则跳转到文件夹内部；否则调用文件管理器的open命令
commands.addCommand(CommandIDs.open, {
  execute: args => {
    const factory = (args['factory'] as string) || void 0;
    const widget = tracker.currentWidget;

    if (!widget) {
      return;
    }

    const { contents } = widget.model.manager.services;
    return Promise.all(
      toArray(
        map(widget.selectedItems(), item => {
          if (item.type === 'directory') {
            const localPath = contents.localPath(item.path);
            return widget.model.cd(`/${localPath}`);
          }

          return commands.execute('docmanager:open', {
            factory: factory,
            path: item.path,
          });
        }),
      ),
    );
  },
  //...
});
```

2.  执行文件管理器的 open 命令(packages/docmanager-extension/src/index.ts 453 行)

```typescript
//解读：根据路径，调用文件管理器的方法去后台获取文件的基本信息，然后调用文件管理器的openOrReveal方法
commands.addCommand(CommandIDs.open, {
  execute: args => {
    const path =
      typeof args['path'] === 'undefined' ? '' : (args['path'] as string);
    const factory = (args['factory'] as string) || void 0;
    const kernel = (args?.kernel as unknown) as Kernel.IModel | undefined;
    const options =
      (args['options'] as DocumentRegistry.IOpenOptions) || void 0;
    return docManager.services.contents
      .get(path, { content: false })
      .then(() => docManager.openOrReveal(path, factory, kernel, options));
  },
  //...
});
```

3.  执行文件管理器的 openOrReveal 方法(packages/docmanager-extension/src/manager.ts)

```typescript
//解读：根据路径和组件类型去查找是否有该组件，有的话则执行打开方法，没有则调用Open方法
//373行
openOrReveal(
    path: string,
    widgetName = 'default',
    kernel?: Partial<Kernel.IModel>,
    options?: DocumentRegistry.IOpenOptions
): IDocumentWidget | undefined {
    const widget = this.findWidget(path, widgetName);
    if (widget) {
    this._opener.open(widget, options || {});
    return widget;
    }
    return this.open(path, widgetName, kernel, options || {});
}

//342行
open(
    path: string,
    widgetName = 'default',
    kernel?: Partial<Kernel.IModel>,
    options?: DocumentRegistry.IOpenOptions
): IDocumentWidget | undefined {
    return this._createOrOpenDocument(
    'open',
    path,
    widgetName,
    kernel,
    options
    );
}
```

4.  执行打开文档方法

    解读：

        1、通过路径以及组件名称获取组件工厂对象，再根据组件工厂对象的模型名称获取到模型工厂对象；
        2、根据路径以及组件工厂名称和内核参数获取内核优先级对象
        3、根据which参数的值，查找和创建上下文对象
        4、创建组件，然后调用open方法打开组件

```typescript
//packages/docmanager-extension/src/manager.ts 534行
private _createOrOpenDocument(
    which: 'open' | 'create',
    path: string,
    widgetName = 'default',
    kernel?: Partial<Kernel.IModel>,
    options?: DocumentRegistry.IOpenOptions
): IDocumentWidget | undefined {
    const widgetFactory = this._widgetFactoryFor(path, widgetName);
    if (!widgetFactory) {
        return undefined;
    }
    const modelName = widgetFactory.modelName || 'text';
    const factory = this.registry.getModelFactory(modelName);
    if (!factory) {
        return undefined;
    }

    // Handle the kernel pereference.
    const preference = this.registry.getKernelPreference(
        path,
        widgetFactory.name,
        kernel
    );

    let context: Private.IContext | null;
    let ready: Promise<void> = Promise.resolve(undefined);

    // Handle the load-from-disk case
    if (which === 'open') {
        // Use an existing context if available.
        context = this._findContext(path, factory.name) || null;
        if (!context) {
            context = this._createContext(path, factory, preference);
            // Populate the model, either from disk or a
            // model backend.
            ready = this._when.then(() => context!.initialize(false));
        }
    } else if (which === 'create') {
        context = this._createContext(path, factory, preference);
        // Immediately save the contents to disk.
        ready = this._when.then(() => context!.initialize(true));
    } else {
        throw new Error(`Invalid argument 'which': ${which}`);
    }

    const widget = this._widgetManager.createWidget(widgetFactory, context);
    this._opener.open(widget, options || {});
    //...
    return widget;
}
```

5. 创建文档的过程

```typescript
//this._createContext(path, factory, preference)

//packages/docmanager-extension/src/manager.ts 454行
private _createContext(
    path: string,
    factory: DocumentRegistry.ModelFactory,
    kernelPreference?: ISessionContext.IKernelPreference
): Private.IContext {
    const adopter = (
        widget: IDocumentWidget,
        options?: DocumentRegistry.IOpenOptions
    ) => {
        this._widgetManager.adoptWidget(context, widget);
        this._opener.open(widget, options);
    };
    const modelDBFactory =
    this.services.contents.getModelDBFactory(path) || undefined;
    const context = new Context({
        opener: adopter,
        manager: this.services,
        factory,
        path,
        kernelPreference,
        modelDBFactory,
        setBusy: this._setBusy,
        sessionDialogs: this._dialogs
    });
    const handler = new SaveHandler({
        context,
        saveInterval: this.autosaveInterval
    });
    Private.saveHandlerProperty.set(context, handler);
    void context.ready.then(() => {
        if (this.autosave) {
            handler.start();
        }
    });
    context.disposed.connect(this._onContextDisposed, this);
    this._contexts.push(context);
    return context;
}

//packages/docregistry/src/context.ts 54行
constructor(options: Context.IOptions<T>) {
    const manager = (this._manager = options.manager);
    this.translator = options.translator || nullTranslator;
    this._trans = this.translator.load('jupyterlab');
    this._factory = options.factory;
    this._dialogs = options.sessionDialogs || sessionContextDialogs;
    this._opener = options.opener || Private.noOp;
    this._path = this._manager.contents.normalize(options.path);
    const localPath = this._manager.contents.localPath(this._path);
    const lang = this._factory.preferredLanguage(PathExt.basename(localPath));

    const dbFactory = options.modelDBFactory;
    if (dbFactory) {
        const localPath = manager.contents.localPath(this._path);
        this._modelDB = dbFactory.createNew(localPath);
        this._model = this._factory.createNew(lang, this._modelDB);
    } else {
        this._model = this._factory.createNew(lang);
    }

    this._readyPromise = manager.ready.then(() => {
        return this._populatedPromise.promise;
    });

    const ext = PathExt.extname(this._path);
    this.sessionContext = new SessionContext({
        sessionManager: manager.sessions,
        specsManager: manager.kernelspecs,
        path: this._path,
        type: ext === '.ipynb' ? 'notebook' : 'file',
        name: PathExt.basename(localPath),
        kernelPreference: options.kernelPreference || { shouldStart: false },
        setBusy: options.setBusy
    });
    this.sessionContext.propertyChanged.connect(this._onSessionChanged, this);
    manager.contents.fileChanged.connect(this._onFileChanged, this);

    const urlResolver = (this.urlResolver = new RenderMimeRegistry.UrlResolver({
        path: this._path,
        contents: manager.contents
    }));
    this.pathChanged.connect((sender, newPath) => {
        urlResolver.path = newPath;
    });
}
```

6. 打开文档过程

```typescript
//packages/docmanager-extension/src/manager.ts 579行
//解读：_opener.open方法是调用的docManager的open方法
const widget = this._widgetManager.createWidget(widgetFactory, context);
this._opener.open(widget, options || {});

//packages/docmanager-extension/src/index.ts 118行
//解读：如果组件未加载，则将组件添加到界面中。然后激活组件，将上下文文档添加到contexts对象中
open: (widget, options) => {
  if (!widget.id) {
    widget.id = `document-manager-${++Private.id}`;
  }
  widget.title.dataset = {
    type: 'document-title',
    ...widget.title.dataset,
  };
  if (!widget.isAttached) {
    app.shell.add(widget, 'main', options || {});
  }
  app.shell.activateById(widget.id);

  // Handle dirty state for open documents.
  const context = docManager.contextForWidget(widget);
  if (context && !contexts.has(context)) {
    if (status) {
      handleContext(status, context);
    }
    contexts.add(context);
  }
};
```

## Notebook 文档渲染过程

1. 页面恢复完成，调用文档初始化方法

```typescript
//packages/docmanager-extension/src/manager.ts 569行
//解读：当页面应用恢复完成之后，调用文档的初始化方法
ready = this._when.then(() => context!.initialize(false));


//packages/docmanager-extension/src/index.ts 142行
//解读: 上面代码中的_when值
const when = app.restored.then(() => void 0);

//packages/docregistry/src/context.ts 227行
//解读： 执行的return this._revert(true)方法
initialize(isNew: boolean): Promise<void> {
    if (isNew) {
        this._model.initialize();
        return this._save();
    }
    if (this._modelDB) {
        return this._modelDB.connected.then(() => {
            if (this._modelDB.isPrepopulated) {
                this._model.initialize();
                void this._save();
                return void 0;
            } else {
                return this._revert(true);
            }
        });
    } else {
        return this._revert(true);
    }
}
```

2. 从服务端下载内容到本地

```typescript
//packages/docregistry/src/context.ts 572行
//解读：1、对后台返回的数据进行反序列化处理
//2、调用this._populate()建立socket通讯
private _revert(initializeModel: boolean = false): Promise<void> {
    //...
    return this._manager.ready
    .then(() => {
        return this._manager.contents.get(path, opts);
    })
    .then(contents => {
        if (this.isDisposed) {
            return;
        }
        const dirty = false;
        if (contents.format === 'json') {
            model.fromJSON(contents.content);
            if (initializeModel) {
                model.initialize();
            }
        } else {
            //...
        }
        this._updateContentsModel(contents);
        model.dirty = dirty;
        if (!this._isPopulated) {
            return this._populate();
        }
    })
    .catch(async err => {
        // ...
    });
}

//packages/notebook/src/model.ts 217行
//解读：将内容进行处理完成之后加入到cells数组中
fromJSON(value: nbformat.INotebookContent): void {
    const cells: ICellModel[] = [];
    const factory = this.contentFactory;
    for (const cell of value.cells) {
    switch (cell.cell_type) {
        case 'code':
        cells.push(factory.createCodeCell({ cell }));
        break;
        case 'markdown':
        cells.push(factory.createMarkdownCell({ cell }));
        break;
        case 'raw':
        cells.push(factory.createRawCell({ cell }));
        break;
        default:
        continue;
    }
    }
    this.cells.beginCompoundOperation();
    this.cells.clear();
    this.cells.pushAll(cells);
    this.cells.endCompoundOperation();
    //...
}

/**
 * 解读:
 * 1、this.cells.pushAll调用了this._cellOrder.push方法；
 * 2、该方法执行了emit，触发了_cellOrder.changed的事件订阅，执行this._onOrderChanged方法；
 * 3、this._onOrderChanged方法执行完成之后，emit出去触发了在model.ts中绑定的事件
 */



//packages/notebook/src/cellList.ts
//34行
constructor(modelDB: IModelDB, factory: NotebookModel.IContentFactory) {
    this._factory = factory;
    this._cellOrder = modelDB.createList<string>('cellOrder');
    this._cellMap = new ObservableMap<ICellModel>();

    this._cellOrder.changed.connect(this._onOrderChanged, this);
}

//333行
pushAll(cells: IterableOrArrayLike<ICellModel>): number {
    const newValues = toArray(cells);
    each(newValues, cell => {
        // Set the internal data structures.
        this._cellMap.set(cell.id, cell);
        this._cellOrder.push(cell.id);
    });
    return this.length;
}

//packages/observables/src/observablelist.ts 439行
//解读：_cellOrder.push方法对应的函数
push(value: T): number {
    const num = this._array.push(value);
    this._changed.emit({
        type: 'add',
        oldIndex: -1,
        newIndex: this.length - 1,
        oldValues: [],
        newValues: [value]
    });
    return num;
}

//packages/notebook/src/model.ts
//78行
constructor(options: NotebookModel.IOptions = {}) {
    //...
    this._cells = new CellList(this.modelDB, this.contentFactory);
    this._trans = (options.translator || nullTranslator).load('jupyterlab');
    this._cells.changed.connect(this._onCellsChanged, this);
    //...
}
//323行
private _onCellsChanged(
    list: IObservableList<ICellModel>,
    change: IObservableList.IChangedArgs<ICellModel>
): void {
    switch (change.type) {
    case 'add':
        change.newValues.forEach(cell => {
            cell.contentChanged.connect(this.triggerContentChange, this);
        });
        break;
    case 'remove':
        break;
    case 'set':
        change.newValues.forEach(cell => {
            cell.contentChanged.connect(this.triggerContentChange, this);
        });
        break;
    default:
        break;
    }
    this.triggerContentChange();
}
```
