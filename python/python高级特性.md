# python 高级特性

### 切片

语法 `tuple`或者`List`  `a`, ` a[startIndex:endIndex]`， 取值范围是`[startIndex,endIndex)`, `startIndex` 为零时可省略 `a[:endIndex]` ,支持倒数切片例如 `a[-2:-1]`

``` python
>>> a = [1,2,3,4]
>>> a[0:3] // 表示从index 0 开始，直到索引3为止
[1,2,3]
>>> L = list(range(100))
[0,1,2....,99]
>>> L[:10]
[0,1,2,3,...,9]

>>> L[-10:]
[90....99]
>>> L[:10:2]  // 前10个数，每两个取一个
[0,2,4,6,8]
>>> L[::5] // 隔五个取一个
>>> [0,5,10,15,....95]
>>> L[:] // 复制List或者元组

```

### 迭代

**list，tuple**

``` python
for x in list:
 .....
```

**`dict`**

默认情况下，`dict`迭代的是key。如果要迭代value，可以用`for value in d.values()`，如果要同时迭代key和value，可以用`for k, v in d.items()`。

如何判断一个对象是可迭代对象呢？方法是通过`collections.abc`模块的`Iterable`类型判断：

```python
>>> from collections.abc import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

Python内置的`enumerate`函数可以把一个`list`变成索引-元素对，这样就可以在`for`循环中同时迭代索引和元素本身：

```python
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print(i, value)
...
0 A
1 B
2 C
// 同时引用两个变量
>>> for x, y in [(1, 1), (2, 4), (3, 9)]:
...     print(x, y)
...
1 1
2 4
3 9
```

### 列表生成式

语法 [表达式 [for 语句,n+] 筛选条件, 可用于有迭代器的数据结构

``` python
a = list(range(10))
b = list(range())
>>> [ str(m) + n for m in a for n in b]
['0a', '0b', '0c', '0d', '0e', '0f', '1a', '1b', '1c', '1d', '1e', '1f', '2a', '2b', '2c', '2d', '2e', '2f', '3a', '3b', '3c', '3d', '3e', '3f', '4a', '4b', '4c', '4d', '4e', '4f', '5a', '5b', '5c', '5d', '5e', '5f', '6a', '6b', '6c', '6d', '6e', '6f', '7a', '7b', '7c', '7d', '7e', '7f', '8a', '8b', '8c', '8d', '8e', '8f', '9a', '9b', '9c', '9d', '9e', '9f']
>>>

>>> d = {'x': 'A', 'y': 'B', 'z': 'C' }
>>> [k + '=' + v for k, v in d.items()]
['y=B', 'x=A', 'z=C']

>>> [x if x % 2 == 0 else -x for x in range(10)]
[0, -1, 2, -3, 4, -5, 6, -7, 8, -9]

>>> [x if x % 2 == 0 else -x for x in range(10) if x > 3]
[4, -5, 6, -7, 8, -9]
```



### 生成器

通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。而且，创建一个包含100万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。

列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。

``` python 
// 第一种方法
g = ( x * x for x in range(10))
>>> next(g)
0
>>> next(g)
1
>>> next(g)
4
...
for x in g:
    print(x)
// 第二种方法
def fib(n):
    i,pre,current=0,0,1
    while i < n:
        yield current
        pre,current = current, pre+current
        i = i + 1
    return 'done'
>>> f = fib(10)
>>> for x in f:
...     print(x)
1
1
2
3
5
8
13
21
34
55

//杨辉三角
def triangles():
    n = 0
    a = [1]
    while True:
        yield a
        b = []
        print(a)
        for i,value in enumerate(a):
            
            print('i=',i)
            if i == 0:
                b.append(a[0])
            else: 
                b.append(a[i-1] + a[i])

            if i == len(a) - 1:
                b.append(a[-1])
            
        a = b
```



### 迭代器

可直接用于 `for` 循环的数据类型有几种

集合类型: `list tuple  set dict str`  等

generator 类型: 包括 **( 列表生成式 )** 以及  ` yield` 的 ` generator function`

可以使用 `isinstance ()` 判断一个对象是否是`Iterable` 对象:

``` python 
from collections.abc import Iterable
>>> isinstance([], Iterable)
>>>  True

>>> isinstance( (x for x in range(10)), Iterable )
```

以上是可迭代对象，那么什么是迭代器呢？

*生成器与迭代器的区别*

生成器不但可以用作与 `for` 循环，还可以被 `next()` 函数不断调用并返回下一个值，直至最后抛出 `StopIteration` 错误表示无法继续返回下一个值

可以被 `next()` 函数调用并不断返回下一个值称为迭代器: Iterator

`list`、`dict`、`str`等`Iterable`变成`Iterator`可以使用`iter()`函数， Python 的`Iterator` 表示的是一个数据流，它通过`next()`函数的调用不断返回下一个数据，知道没有数据抛出 `StopIteration`错误。可以把这个数据流看作有序序列n，我们不能提前知道序列的长度，只能不断通过 `next()` 函数实现按需计算