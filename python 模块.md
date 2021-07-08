# 模块

### 概念

python 中一 `.py` 文件称为一个模块

为了避免模块名称冲突，python 引入按文件目录来组织模块代码的方法，称为包(package), 如 `abc.py`  与 `xyz.py` 两个模块与其他模块冲突了，我们可以通过一个顶层包名来组织

```ascii
mycompany
├─ __init__.py
├─ abc.py
└─ xyz.py
```

没一个包目录下都会有一个`__init__.py` 文件，这个文件是必须存在的，否则 Python 会把该目录当成普通文件目录，`__init__.py`可以是空文件，也可以有Python代码，因为`__init__.py`本身就是一个模块，而它的模块名就是`mycompany`。

```ascii
mycompany
 ├─ web
 │  ├─ __init__.py
 │  ├─ utils.py
 │  └─ www.py
 ├─ __init__.py
 ├─ abc.py
 └─ utils.py
```

文件`www.py`的模块名就是`mycompany.web.www`，两个文件`utils.py`的模块名分别是`mycompany.utils`和`mycompany.web.utils`。

### 模块的使用

模块的标准格式

``` python
#!/usr/bin/env python3  //linux环境声明可作为脚本执行
# -*- coding: utf-8 -*-  

' a test module '  // doc

__author__ = 'Michael Liao' 

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__': // 当作为脚本运行时，执行test方法
    test()
```

作用域

_前缀标记希望内部使用

`__xxxx__`表示特殊变量，有特殊用途

之所以我们说，private函数和变量“不应该”被直接引用，而不是“不能”被直接引用，是因为Python并没有一种方法可以完全限制访问private函数或变量，但是，从编程习惯上不应该引用private函数或变量，python 的作用域是采用约定的方式

### 安装常用模块

一般使用 `anaconda` ，是一个基于python 的数据处理和科学计算平台，内置很多三方库，否则手动使用 `pip install packageName` 来安装



###模糊搜索路径

默认情况下，Python解释器会搜索当前目录、所有已安装的内置模块和第三方模块，搜索路径存放在`sys`模块的`path`变量中：

``` python
>>> import sys
>>> sys.path
['', 'D:\\ProgramData\\Miniconda3\\python38.zip', 'D:\\ProgramData\\Miniconda3\\DLLs', 'D:\\ProgramData\\Miniconda3\\lib', 'D:\\ProgramData\\Miniconda3', 'D:\\ProgramData\\Miniconda3\\lib\\site-packages', 'D:\\ProgramData\\Miniconda3\\lib\\site-packages\\pip-21.0-py3.8.egg', 'd:\\workspace\\opensourceproject\\jupyter_extensions\\algorithm-images', 'd:\\workspace\\opensourceproject\\jupyterlab', 'D:\\ProgramData\\Miniconda3\\lib\\site-packages\\win32', 'D:\\ProgramData\\Miniconda3\\lib\\site-packages\\win32\\lib', 'D:\\ProgramData\\Miniconda3\\lib\\site-packages\\Pythonwin']

```

python 有两种方法添加指定路径模块

- `sys.path.append('path/to/module')` 
- 配置环境变量` PYTHONPATH`, 该环境变量下的内容会自动添加至模块搜索路径中 ，设置方式与设置Path环境变量类似。注意只需要添加你自己的搜索路径，Python自己本身的搜索路径不受影响。

