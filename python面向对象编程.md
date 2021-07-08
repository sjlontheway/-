# Python 面向对象编程



面向对象编程——Object Oriented Programming，简称OOP，是一种程序设计思想。OOP把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。

面向过程的程序设计把计算机程序视为一系列的命令集合，即一组函数的顺序执行。为了简化程序设计，面向过程把函数继续切分为子函数，即把大块函数通过切割成小块函数来降低系统的复杂度

### 类和实例

*python* 定义类是通过class 关键字来定义的 ，`class`后面紧接着是类名，即以下示例中`Student`，类名通常是大写开头的单词，紧接着是`(object)`，表示该类是从哪个类继承下来的，继承的概念我们后面再讲，通常，如果没有合适的继承类，就使用`object`类，这是所有类最终都会继承的类

``` python
class Student(object):
    def __init__(self,name,score):
        self.name = name
        self.score = score
        
    def print_score(self):
        print(f'{self.name} get score: {self.score}')
        
>>> student = Student('Tom','98')
>>> student.print_score()
Tom get score: 98
```

### 数据封装

根据代码示例来看，`__init__` 方法相当于其他语言中的构造函数，可以将对象的实例属性传进去，构造新的对象实例，注意在实例化的时候我们不需要传`self`参数

### 访问限制

我们知道 Class 内部有属性和方法能够被外部直接调用，但如果有私有的内部方法和属性不希望被外部访问时，我们可以在属性的名称前加两个下划线`__` ，实例的变量名如果以`__`开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问，这样就确保了外部代码不能随意修改对象内部的状态，这样通过访问限制的保护，代码更加健壮。

可以通过设置内部访问器方法来对内部属性进行读取与修改

实际上python解释器会把`__`开头的属性重命名成 `_类名__属性名` 或其他，取决于不同解释器对名称的修改

些时候，你会看到以一个下划线开头的实例变量名，比如`_name`，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。



### 继承与多态

这就是著名的“开闭”原则：

对扩展开放：允许新增`Animal`子类；

对修改封闭：不需要修改依赖`Animal`类型的`run_twice()`等函数。

``` python
class Animal(object):
    def run(self):
        print('animal is running!')
class Dog(Animal):
    def run(self):
        print('Dog is running!')
class Cat(Animal):
    def run(self):
        print('Cat is running!')
        
def run_twice(animal):
    animal.run()
    animal.run()
>>> run_twice(Animal())
>>> run_twice(Cat())

class Tortoise(Animal):
    def run(self):
        print('Tortoise is running!')
>>> run_twice(Tortoise())


```



```ascii
                ┌───────────────┐
                │    object     │
                └───────────────┘
                        │
           ┌────────────┴────────────┐
           │                         │
           ▼                         ▼
    ┌─────────────┐           ┌─────────────┐
    │   Animal    │           │    Plant    │
    └─────────────┘           └─────────────┘
           │                         │
     ┌─────┴──────┐            ┌─────┴──────┐
     │            │            │            │
     ▼            ▼            ▼            ▼
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│   Dog   │  │   Cat   │  │  Tree   │  │ Flower  │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
```

### 静态语言Vs动态语言

对于静态语言（例如Java）来说，如果需要传入`Animal`类型，则传入的对象必须是`Animal`类型或者它的子类，否则，将无法调用`run()`方法。

对于Python这样的动态语言来说，则不一定需要传入`Animal`类型。我们只需要保证传入的对象有一个`run()`方法就可以了：

```
class Timer(object):
    def run(self):
        print('Start...')
```

这就是动态语言的“鸭子类型”，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。

Python的“file-like object“就是一种鸭子类型。对真正的文件对象，它有一个`read()`方法，返回其内容。但是，许多对象，只要有`read()`方法，都被视为“file-like object“。许多函数接收的参数就是“file-like object“，你不一定要传入真正的文件对象，完全可以传入任何实现了`read()`方法的对象。

### duck type

由于动态语言没有对参数进行类型检查，运行时，只需要判断对象是否拥有该方法或者属性即可。

***继承可以把父类的所有功能都直接拿过来，这样就不必重零做起，子类只需要新增自己特有的方法，也可以把父类不适合的方法覆盖重写。动态语言的鸭子类型特点决定了继承不像静态语言那样是必须的。***

### **获取对象信息**

拿到一个对象时如何判断对象的类型、有哪些方法？

##### 使用`type()`

基本类型都可以使用`type()` 来进行判断 

``` python
>>> import types
>>> def fn():
...     pass
...
>>> type(fn)==types.FunctionType
True
>>> type(abs)==types.BuiltinFunctionType
True
>>> type(lambda x: x)==types.LambdaType
True
>>> type((x for x in range(10)))==types.GeneratorType
True
```

##### 使用instance()

`isinstance()`判断的是一个对象是否是该类型本身，或者位于该类型的父继承链上。并且还可以判断一个变量是否是某些类型中的一种，比如下面的代码就可以判断是否是list或者tuple：

```python
>>> isinstance('a', str)
True
>>> isinstance(123, int)
True
>>> isinstance(b'a', bytes)
True
>>> isinstance([1, 2, 3], (list, tuple))
True
>>> isinstance((1, 2, 3), (list, tuple))
True
```



##### 使用dir()

可以使用该方法获取对象的所有属性和方法

类似`__xxx__`的属性和方法在Python中都是有特殊用途的，比如`__len__`方法返回长度。在Python中，如果你调用`len()`函数试图获取一个对象的长度，实际上，在`len()`函数内部，它自动去调用该对象的`__len__()`方法，所以，下面的代码是等价的：

``` python 
>>> class MyDog(object):
...     def __len__(self):
...         return 100
...
>>> dog = MyDog()
>>> len(dog)
100
```

仅仅把属性和方法列出来是不够的，配合`getattr()`、`setattr()`以及`hasattr()`，我们可以直接操作一个对象的状态：

```python
>>> class MyObject(object):
...     def __init__(self):
...         self.x = 9
...     def power(self):
...         return self.x * self.x
...
>>> obj = MyObject()
>>> hasattr(obj, 'x') # 有属性'x'吗？
True
>>> obj.x
9
>>> hasattr(obj, 'y') # 有属性'y'吗？
False
>>> setattr(obj, 'y', 19) # 设置一个属性'y'
>>> hasattr(obj, 'y') # 有属性'y'吗？
True
>>> getattr(obj, 'y',404) # 获取属性'y'
19
>>> obj.y # 获取属性'y'
19
>>> getattr(obj, 'z',404) # 获取属性'z',没有返回默认值
404
```

##### 实例属性与类属性

实例属性属于各个实例所有，互不干扰；

类属性属于类所有，所有实例共享一个属性；

不要对实例属性和类属性使用相同的名字，否则将产生难以发现的错误。