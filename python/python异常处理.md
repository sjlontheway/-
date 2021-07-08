# python 异常处理

### 错误处理异常捕获

使用形式如下：

``` python
try:
    # code
except ValueError as e:
    # error handler
finally:
    # finally code logic
```

### 调试

##### 直接print打印

##### 断言

凡是可以使用`print()` 来辅助查看的地方，都可以用断言(`assert`) 来替代，使用如下：

``` python
def foo(s):
    n = int(s)
    assert s != 0,'n is zero!'
    return 10 / n
def main():
    foo('0')

# $ python err.py
# Traceback (most recent call last):
# ...
# AssertionError: n is zero!
```

可以在-O来启动程序，屏蔽断言

##### logging 来记录错误

##### `pdb` 来断点调试

##### `pdb.set_trace()`

这个方法也是用`pdb`，但是不需要单步执行，我们只需要`import pdb`，然后，在可能出错的地方放一个`pdb.set_trace()`，就可以设置一个断点：

##### `IDE`

如果要比较爽地设置断点、单步执行，就需要一个支持调试功能的IDE。目前比较好的Python IDE有：

Visual Studio Code：https://code.visualstudio.com/，需要安装Python插件。

PyCharm：http://www.jetbrains.com/pycharm/



### 单元测试

### 文档测试



