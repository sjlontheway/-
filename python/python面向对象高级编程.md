# 面向对象高级编程

[TOC]

###  使用`__slot__`

动态语言允许我们在运行时，给实例绑定属性以及方法，也允许直接给class 绑定方法

如果我们需要限制实例属性该如何处理，那么我们将用到`__slot__` ，比如我们值=只允许一个类加入指定属性

``` python
class Student(object):
    __slot__ = ('name','age')
    
>>> s = Student() # 创建新的实例
>>> s.name = 'Michael' # 绑定属性'name'
>>> s.age = 25 # 绑定属性'age'
>>> s.score = 99 # 绑定属性'score'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
```



*`__slot __`* 只对当前对象实例产生作用，不会对继承的子类产生作用 

### 使用@property

`property`的实现比较复杂，我们先考察如何使用。把一个getter方法 `score()`变成属性，只需要加上`@property`就可以了，此时，`@property`本身又创建了另一个装饰器`@score.setter`，负责把一个setter方法变成属性赋值，于是，我们就拥有一个可控的属性操作：

### 多重继承

通过多重继承，一个子类可以同时获得多个父类的所有功能

### `MixIn`

`MixIn`的目的就是给一个类增加多个功能，在设计类的时候我们优先考虑多重继承来组合多个`MixIn`的功能，而不是设计多层次且复杂的继承关系

由于Python允许使用多重继承，因此，`MixIn`就是一种常见的设计。

只允许单一继承的语言（如Java）不能使用`MixIn`的设计。

### 定制类

 `__str()__` 重写实例对象 `print`输出

`__repr()__` 重写对象字符串输出如shell 控制台，调试 实例对象

`__iter()__` 迭代器属性

`__next()__`  获取迭代元素

`__getitem()__` 按下标索引获取元素

`__getattr__` 正常情况下，当我们调用类的方法或属性时，如果不存在，就会报错，写一个`__getattr__()`方法，动态返回一个属性，当调用不存在的属性时，比如`score`，Python解释器会试图调用`__getattr__(self, 'score')`来尝试获得属性

`__call__`   实例可以 s = Student() ,s() 调用的就是此方法 可通过callable方法判断是否可以调用



### Enum 枚举

### 元类



动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的。

#### type()

当Python解释器载入`hello`模块时，就会依次执行该模块的所有语句，执行结果就是动态创建出一个`Hello`的class对象

``` 
class Hello(object):
    def hello(self, name='world'):
        print('Hello, %s.' % name)
        
>>> from hello import Hello
>>> h = Hello()
>>> h.hello()
Hello, world.
>>> print(type(Hello))
<class 'type'>
>>> print(type(h))
<class 'hello.Hello'>
```

`type()` 可以动态定义`class`

要创建一个class对象，`type()`函数依次传入3个参数：

1. class的名称；
2. 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
3. class的方法名称与函数绑定，这里我们把函数`fn`绑定到方法名`hello`上。

正常情况下，我们都用`class Xxx...`来定义类，但是，`type()`函数也允许我们动态创建出类来，也就是说，动态语言本身支持运行期动态创建类，这和静态语言有非常大的不同，要在静态语言运行期创建类，必须构造源代码字符串再调用编译器，或者借助一些工具生成字节码实现，本质上都是动态编译，会非常复杂。

### 元类`mataclass`

控制类的创建行为

当我们定义了类以后，就可以根据这个类创建出实例，所以：先定义类，然后创建实例。

但是如果我们想创建出类呢？那就必须根据`metaclass`创建出类，所以：先定义`metaclass`，然后创建类。连接起来就是：先定义`metaclass`，就可以创建类，最后创建实例。