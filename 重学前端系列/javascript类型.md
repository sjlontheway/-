# JavaScript 类型

### **？关于类型的几个问题**

- 为什么有的编程规范要求用void 0 代替 undefined？
- 字符串有最大长度吗？
- 0.1+0.2 在js中为什么不等于0.3？
- ES6 新加入的Symbol是什么东西？
- 为什么给对象添加的方法能用在基本类型上？



### JS 类型

- Undefined
- Null
- Boolean
- String
- Number
- Symbol
- Object

#### Undefined、Null

**定义**

- **Null** : The Null type has exactly one value, called null.
- **Undefined**:  The Undefined type has exactly one value, called undefined. Any variable that has not been assigned a value has the value undefined.

首先我们了解下Undefined代表的含义,Undefined 类型表示未定义，他的类型只有一个值，就是undefined。 任何变量在赋值前都是Undefined 类型、值为undefined，一般我们是用名为undefined 全局变量来表达这个值，或者用void 运算来把任意一个表达式变成 undefined值。 

**由于Javascript的语言设计 undefined 是一个变量，而不是关键字，我们避免无意中被篡改，于是有代码规范使用 void 0 来获取 undefined 值;**

Null 与 Undefined 虽然在真值判断中都表示false，但其语义是不同的，Null表示的是变量定义了但是值为空。我们一般不会把变量赋值为undefined，所以通过Undefined来表示未初始化过的变量。

Null 类型只有一个值，就是null，它的语义表示空值，并且null是关键字，所以我们可以安全的使用null关键字来获取null值。 

### String

String 表示文本数据。最大长度是 2^53 - 1 ，这在一般开发中是够用的，字符的最大长度是针对的UTF-16编码，javaScript把utf16的一个单元当作一个字符来处理

### Number

Javascript Number是双精度浮点型，number类型运算都要想将其转化为二进制，将二进制运算，运算的结果再转化为十进制，因为number是64位双精度，小数部分只有52位，但0.1转化成为二进制是无限循环的，所以四舍五入了，这里就发生了精度丢失，0.1的二进制和0.2的二进制相加需要保留有效数字，所以又发生了精度丢失，所以结果为0.300000000000004，所以为false，而0.2+0.3恰好两个转化成为二进制和相加的过程都不会发生精度丢失，所以为true，如果你看不懂推荐看看这个教程：https://www.taopoppy.cn/Front-end/javascript_primitiveType.html#number-%E7%B1%BB%E5%9E%8B

JavaScript中的Number类型有 18437736874454810627(即2^64-2^53+3) 个值。

JavaScript 中的 Number 类型基本符合 IEEE 754-2008 规定的双精度浮点数规则，但是JavaScript为了表达几个额外的语言场景（比如不让除以0出错，而引入了无穷大的概念），规定了几个例外情况：

- NaN，占用了 9007199254740990(2 ^53)，这原本是符合IEEE规则的数字；
- Infinity，无穷大；
- -Infinity，负无穷大。 另外，值得注意的是，JavaScript中有 +0 和 -0，在加法类运算中它们没有区别，但是除法的场合则需要特别留意区分，“忘记检测除以-0，而得到负无穷大”的情况经常会导致错误，而区分 +0 和 -0 的方式，正是检测 1/x 是 Infinity 还是 -Infinity。



### Symbol

**定义：** The Symbol type is the set of all non-String values that may be used as the key of an Object property (6.1.7).Each possible Symbol value is unique and immutable.Each Symbol value immutably holds an associated value called [[Description]] that is either undefined or a String value.

Symbol 是ES6中引入的新类型，它是一切非字符串的对象key的集合，在ES6 规范中，整个对象系统被用 Symbol 重构,下面是表示使用 Symbol.interator 来定义for ... of 在对象上的行为:

``` js
    var o = new Object

    o[Symbol.iterator] = function() {
        var v = 0
        return {
            next: function() {
                return { value: v++, done: v > 10 }
            }
        }        
    };

    for(var v of o) 
        console.log(v); // 0 1 2 3 ... 9
```

### Object





### 装箱转换

每一种基本类型 Number、String、Boolean、Symbol 在对象中都有对应的类，所谓装箱转换，正是把基本类型转换为对应的对象，它是类型转换中一种相当重要的种类。

未弄懂问题：

- js 精度问题，双精度浮点型如何表示
- 装箱、拆箱机制