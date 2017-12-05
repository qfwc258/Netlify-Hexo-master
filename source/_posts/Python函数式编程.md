title: Python函数
date: 2017-12-5
comments: true
tags: 
 - Python
categories: Python
----------

函数是Python内建支持的一种封装，我们通过把大段代码拆成函数，通过一层一层的函数调用，就可以把复杂任务分解成简单的任务，这种分解可以称之为面向过程的程序设计。函数就是面向过程的程序设计的基本单元。

而函数式编程（请注意多了一个“式”字）——Functional Programming，虽然也可以归结到面向过程的程序设计，但其思想更接近数学计算。

我们首先要搞明白计算机（Computer）和计算（Compute）的概念。

在计算机的层次上，CPU执行的是加减乘除的指令代码，以及各种条件判断和跳转指令，所以，汇编语言是最贴近计算机的语言。

而计算则指数学意义上的计算，越是抽象的计算，离计算机硬件越远。

对应到编程语言，就是越低级的语言，越贴近计算机，抽象程度低，执行效率高，比如C语言；越高级的语言，越贴近计算，抽象程度高，执行效率低，比如Lisp语言。

函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

Python对函数式编程提供部分支持。由于Python允许使用变量，因此，Python不是纯函数式编程语言。

<!-- more -->

## 高阶函数

Higher-order function

### 变量可以指向函数

以Python内置的求绝对值的函数`abs()`为例，调用该函数用以下代码：

```python
>>> abs(-10)
10
```

但是，如果只写`abs`呢？

```python
>>> abs
<built-in function abs>
```

可见，`abs(-10)`是函数调用，而`abs`是函数本身。

要获得函数调用结果，我们可以把结果赋值给变量：

```python
>>> x = abs(-10)
>>> x
10
```

但是，如果把函数本身赋值给变量呢？

```python
>>> f = abs
>>> f
<built-in function abs>
```

结论：函数本身也可以赋值给变量，即：变量可以指向函数。

如果一个变量指向了一个函数，那么，可否通过该变量来调用这个函数？用代码验证一下：

```python
>>> f = abs
>>> f(-10)
10
```

成功！说明变量`f`现在已经指向了`abs`函数本身。直接调用`abs()`函数和调用变量`f()`完全相同。

### 函数名也是变量

函数名是什么呢？函数名其实就是指向函数的变量！对于`abs()`这个函数，完全可以把函数名`abs`看成变量，它指向一个可以计算绝对值的函数！

如果把`abs`指向其他对象，会有什么情况发生？

```python
>>> abs = 10
>>> abs(-10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not callable
```

把`abs`指向`10`后，就无法通过`abs(-10)`调用该函数了！因为`abs`这个变量已经不指向求绝对值函数而是指向一个整数`10`！

当然实际代码绝对不能这么写，这里是为了说明函数名也是变量。要恢复`abs`函数，请重启Python交互环境。

注：由于`abs`函数实际上是定义在`import builtins`模块中的，所以要让修改`abs`变量的指向在其它模块也生效，要用`import builtins; builtins.abs = 10`。

### 传入函数

既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

一个最简单的高阶函数：

```python
def add(x, y, f):
    return f(x) + f(y)
```

当我们调用`add(-5, 6, abs)`时，参数`x`，`y`和`f`分别接收`-5`，`6`和`abs`，根据函数定义，我们可以推导计算过程为：

```python
x = -5
y = 6
f = abs
f(x) + f(y) ==> abs(-5) + abs(-6) ==> 11
return 11
```

### map/reduce

Python内建了`map()`和`reduce()`函数

先来看`map()`,`map()`函数接收两个参数,一个是函数,一个是`Iterable`,`map`将传入的函数依次作用到序列的每个元素,并把结果作为新的`Iterator`返回.

例: 有一个函数`f(x)=x²`，要把这个函数作用在一个list `[1, 2, 3, 4, 5, 6, 7, 8, 9]`上，就可以用map()实现如下：

```python
def f(x):
  return x * x

r = map(f[1, 2, 3, 4, 5, 6, 7, 8, 9])
print(list(r))
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

`map`传入的第一个参数是`f`,即函数对象本身. 由于结果`r`是一个`Iterator`,`Iterator`是惰性序列,因此通过`list()`函数让它把整个序列都计算出来并返回一个list.

你可能会想,不需要`map()`函数,写一个循环,一样可计算出结果:

```python
L = []
for n in [1, 2, 3, 4, 5, 6, 7, 8, 9]:
    L.append(f(n))
print(L)
```    

的确可以,但是从上面的循环代码,无法一眼看明白把`f(x)`作用在list的每一个元素并把结果生成一个新的`list`

所以,`map`作为高阶函数,事实上它把运算规则抽取了,因此,我们不但可以计算简单的`f(x) = x²`,还可以计算任意复杂的函数,比如,把这个list所有的数字转为字符串

```python
print(list(map(str,[1, 2, 3, 4, 5, 6, 7, 8, 9])))
['1', '2', '3', '4', '5', '6', '7', '8', '9']
```

只需要一行代码.

reduce的用法.

reduce把一个函数作用在一个序列`[x1, x2, x3, ...]`上,这个函数必须接收两个参数,`recude`把结果继续和序列的下一个元素做累积计算:

```python
reduce(f, [x1, x3, x3, x4]) = f(f(f(x1, x2), x3), x4)
```

比如说对象一个序列求和,就可以用`reduce`实现:

```python               
from functools import reduce
def add(x, y):
    return x + y
print(reduce(add, [1, 3, 5, 7, 9]))
25
```

当然求和运算可以直接用Python内建函数`sum()`，没必要动用`reduce`。

但是如果要把序列`[1, 3, 5, 7, 9]变`换成整数`13579`，`reduce`就可以派上用场：

```python
from functools import reduce
def fn(x, y):
    return x * 10 + y
print(reduce(fn, [1, 3, 5, 7, 9]))
```

这个例子本身没多大用处,但是,如果考虑到字符串`str`也是一个序列,对上面的例子稍加改，配合`map()`，我们就可以写出把`str`转换为`int`的函数：

```Python
from functools import reduce
def fn(x, y):
    return x * 10 + y

def char2num(s):
  digits = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}
  return digits[s]
Reduce(fn, map(char2num, '13579'))
13579
```

整理成一个str2int的函数就是：

```Python
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def str2int(s):
    def fn(x, y):
        return x * 10 + y
    def char2num(s):
        return DIGITS[s]
    return reduce(fn, map(char2num, s))
```

还可以用lambda函数进一步简化成：

```python
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def char2num(s):
    return DIGITS[s]

def str2int(s):
    return reduce(lambda x, y: x * 10 + y, map(char2num, s))
```

也就是说，假设Python没有提供`int()`函数，你完全可以自己写一个把字符串转化为整数的函数，而且只需要几行代码！