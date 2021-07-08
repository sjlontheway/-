# 函数式编程

[TOC]

### 高阶函数

变量可以指向函数

函数名也是变量

```  python
def add(x, y, f):
    return f(x) + f(y)

>>> print(add(-5, 6, abs))
11

abs = 10
>>> abc
10
>>> abs(10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not callable
```

### map/reduce

`map(f, Iterable)`

`reduce(f, Iterable)` // return  map

`filter()`

 `sorted( Iterable,key=None,reverse=false)`   key 可以用预处理数据，比如`lower、 abs` 等，可以选择参数是否逆序

`__name__`





**__name__ 是当前模块名，当模块被直接运行时模块名为 __main__ 。这句话的意思就是，当模块被直接运行时，以下代码块将被运行，当模块是被导入时，代码块不被运行。**

### 返回函数

在Python中，如果变量仅仅是被引用而没有被赋值过，那么默认被视作全局变量。如果一个变量在函数中**被赋值过**，那么就被视作局部变量。

**闭包**

``` python
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax= ax+n
        return ax
    return sum

>> f = lazy_sum(1,2,3)
>>> f 
<function lazy_sum.<locals>.sum at xxxx>
>>f()
6
```

### 匿名函数

`lambda *args:  return `

### 装饰器

函数也是一个对象，函数对象可以被赋值给变量，通过变量也能调用函数

### 偏函数

类似 `js` 柯里化,Python的`functools`模块提供了很多有用的功能，其中一个就是偏函数（Partial function）。要注意，这里的偏函数和数学意义上的偏函数不一样。 注意一点，默认参数是从左到右，当我们固定住中间一个参数时，比如`def  fun(a,b,c):` 函数，固定住 `b` 的值，那么在使用的时候，`b`以及`c`参数将变成`**kw`形式

``` python
>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
>>> int2('1010101')
85
```

