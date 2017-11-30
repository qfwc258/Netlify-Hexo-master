title: Python函数
date: 2017-11-30 11:14:22
comments: true
tags: 
 - Python
categories: Python
----------

## 调用函数

Python内置了很多有用的函数，可供我们直接调用。

要调用一个函数，需要知道函数的名称和参数，比如求绝对值的函数`abs`，只有一个参数，[参考Python官方网站文档](https://docs.python.org/3/library/functions.html#abs)

也可以在交互式命令行通过`help(abs)`查看`abs`函数的帮助信息。

调用`abs`函数：

```python
>>> abs(100)
100
>>> abs(-20)
20
>>> abs(12.34)
12.34
```
<!-- more -->

调用函数的时候，如果传入的参数数量不对，会报错`TypeError`，并且Python会明确的告诉你：`abs()`有且仅有`1`个参数，但你给出了`n`个：

```python
>>> abs(1,2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: abs() takes exactly one argument (2 given)
```

如果传入的参数数量是对的，但参数类型不能被函数所接受，也会报错`TypeError`，并且给出错误信息：`str`是错误的参数类型：

```python
>>> abs('a')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: bad operand type for abs(): 'str'
```

而`max`函数`max()`可以接收任意多个参数，并返回最大的那个：

```python
>>> max(1,2)
2
>>> max(3,1,2,-1)
3
```

### 数据类型转换

Python内置的常用函数还包括数据类型转换函数，比如`int()`函数可以把其它数据类型转换为整数：

```python
>>> int('123')
123
>>> int(12.34)
12
>>> float('12.34')
12.34
>>> str(1.23)
'1.23'
>>> str(100)
'100'
>>> bool(1)
True
>>> bool('')
False
```

函数名其实就是指向一个函数对象的引用，完全可以把函数名赋值给一个变量，相当于给这个函数起了一个别名：

```python
>>> a = abs # 变量a指向abs函数
>>> a(-1) # 所以也可以通过a调用abs函数
1
```

`hex()`函数把一个整数转换成十六进制表示的字符串：

```python
>>> hex(1)
0x1
```

### 小结

调用Python函数，需要根据函数定义，传入正确的参数。如果函数调用出错，一定要学会看错误信息(英文很重要)。

----

## 定义函数

在Python中，定义一个函数需要使用`def`语句，依次写出函数名、括号、括号中的参数和冒号`：`，然后，在缩进块中编写函数体，函数的返回值用`return`语句返回。

自定义一个求绝对值的`my_abs`函数：

```python
def my_abs(x):
  if x >= 0:
    return x
  else:
    return -x
  
print(my_abs(-99))

99
```

值得注意的是，函数体内部的语句在执行时，一旦执行到`return`时，函数就执行完毕，并将结果返回。因此，函数内部通过条件判断和循环可以实现非常复杂的逻辑。

如果没有`return`语句，函数执行完毕也会返回结果，只是结果为`None`，`return None`可以简写为`return`。

在Python交互环境中定义函数时，主意Python会出现`...`的提示。函数定义结束后需要按两次回车重新回到`>>>`提示符下：

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171130101706.jpg)

如果已经把'my_abs()'的函数定义保存为`abstest.py`文件，那么，可以直接在这个文件的目录下启动Python解释器，用`from abstest import my_abs`来导入'my_abs()'函数。 **注意：`abstest`是文件名(不包含.py扩展名)**

### 空函数

如果想定义一个什么事也不做的空函数，可以用`pass`语句：

```python
def nop():
  pass
```

`pass`语句什么都不做，那有什么用？ 实际上`pass`语句可用作占位符，比如还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。

`pass`还可以放在其他语句里，比如：

```python
if age >= 18:
  pass
```

缺少了`pass`，代码运行就会有语法错误。

### 参数检查

调用函数时，如果参数个数不对，Python解释器会自动检查出来，并抛出`TypeError`：

```python
>>> my_abs(1,2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: my_abs() takes 1 positional argument but 2 were given
```

但是如果参数类型不对，Python解释器就无法帮我们检查。试试`my_abs`和内置函数`abs`的差别：

```python
>>> my_abs('A')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in my_abs
TypeError: '>=' not supported between instances of 'str' and 'int'
>>> abs('A')
Traceback (most recent call last):
  File "<stdin>", line 7, in <module>
TypeError: bad operand type for abs(): 'str'
```

当传入了不恰当的参数时，内置函数`abs`会检查出参数错误，而我们定义的`my_abs`没有参数检查，会导致`if`语句出错，出错信息和`abs`不一样。所以，这个函数不够完善。

修改一下`my_abs`函数的定义，对参数类型做检查，只允许整数和浮点数类型的参数。数据类型检查可以用内置函数`isinstance()`实现：

```python
def my_abs(x):
    if not isinstance(x,(int,float)):
        raise TypeError('bad operand type')
    if x >= 0:
        return x
    else:
        return -x
```

添加了参数类型后，如果传入错误的参数类型，函数就可以抛出一个错误：

```python
>>> my_abs('A')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in my_abs
TypeError: bad operand type
```

### 返回多个值

在游戏中经常需要从一个点移动到另一个点，给出坐标、位移和角度，就可以计算出新的坐标：

```python
import math

def move(x, y, step, angle = 0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx,ny
```

`import math`语句表示导入`math`包，并允许后续代码引用`math`包里的`sin`、`cos`等函数。

```python
>>> x, y = move(100, 100, 60, math.pi / 6)
>>> print(x, y)
151.96152422706632 70.0
```

但其实这只是一种假象，Python返回的仍然是单一值：

```python
>>> r = move(100, 100,  60, math.pi / 6)
>>> print(r)
(151.96152422706632, 70.0)
```

这样就能清楚的看到，返回值是一个tuple！但是，在语法上，返回一个tuple可以省略括号，而多个变量可以同时接受一个tuple，按位置赋给对应的值，所以，Python的函数返回多值其实就是返回一个tuple，但写起来更方便。

### 练习

定义一个函数`quadratic(a, b, c)`，接收3个参数，返回一元二次方程： `ax² + bx + c = 0` 的两个解(计算平方根可以调用`math.sqrt()`函数)：

```python
import math

def quadratic(a, b, c):
    for i in (a, b, c):
        if not isinstance(i, (float, int)):
            raise TypeError('数据类型输入有误，请重新输入！')
    delta = b * b - 4 * a * c
    n=-b / (2 * a)
    if a == 0:
        print('此方程不是一元二次方程!')
    elif delta == 0:
        print('该一元二次方程只有一个根:',n)
        return n
    elif delta < 0:
        print('该一元二次方程没有解!')
    elif delta > 0:
        x1 = (-b + math.sqrt(delta)) / (2*a)
        x2 = (-b - math.sqrt(delta)) / (2*a)
        return x1, x2

# 测试:
print('quadratic(2, 3, 1) =', quadratic(2, 3, 1))
print('quadratic(1, 3, -4) =', quadratic(1, 3, -4))

if quadratic(2, 3, 1) != (-0.5, -1.0):
    print('测试失败')
elif quadratic(1, 3, -4) != (1.0, -4.0):
    print('测试失败')
else:
    print('测试成功')
```

### 小结

定义函数时，需要确定函数名和参数个数

如果有必要，可以先对参数的数据类型做检查

函数体内部可以用`return`随时返回函数结果

函数执行完毕也没有`return`语句时，自动`return None`

函数可以同时返回多个值，但其实就是一个tuple

---

## 函数的参数

定义函数的时候，需要把参数的名字和位置确定下来，函数的接口定义就完成了。对于函数的调用者来说，只需要知道如何传递正确的参数，以及函数将返回什么样的值就够了，函数内部的复杂逻辑被封装起来，调用者无需了解。

Python的函数定义非常简单，但灵活度却非常强大。除了正常定义的必选参数外，还可以使用默认参数、可变参数和关键字参数，使得函数定义出来的接口，不但能处理复杂的参数，还可以简化调用者的代码。

### 位置参数

我们先写一个计算x²的函数：

```python
def power(x):
  return x * x
```

对于`power(x)`函数，参数`x`就是一个位置参数。

当我们调用`power`函数时，必须传入有且仅有的一个参数`x`:

```python
>>> power(5)
25
>>> power(15)
225
```

现在，如果我们要计算`x³`怎么办？ 可以再定义一个`power3`函数，但是如果要计算x的4次方、x的5次方...怎么办？ 不可能定义无限多个函数。

所以，可以把`power(x)`改为`power(x,n)`，用来计算x的n次方：

```python
def power(x, n):
  s = 1
  while n > 0:
    n = n - 1
    s = s * x
  return s
```

对于这个修改后的`power(x, n)`函数，可以计算任意n次方：

```python
>>> power(5, 2)
25
>>> power(5, 3)
125
```

修改后的`power(x, n)`函数有两个参数：`x`和`n`，这两个参数都是位置参数，调用函数时，传入的两个值按照位置顺序依次赋给参数`x`和`n`。

### 默认参数

新的`power(x, n)`函数定义没有问题，但是，旧的调用代码失败了，原因是我们增加了一个参数，导致旧的代码因为缺少一个参数而无法正常调用：

```python
>>> power(5)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: power() missing 1 required positional argument: 'n'
```

Python的错误信息很明确，调用函数`power()`缺少了一个位置参数`n`

这个时候，默认参数就派上用场了。由于我们经常计算x²，所以，完全可以把第二个参数n的默认值设为2：

```python
def power(x, n = 2):
  s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```

这样，当我们调用`power(5)`的时候，就相当于调用了`power(5, 2)`：

```python
>>> power(5)
25
>>> power(5, 2)
25
```

而对于 `x > 2`的其他情况，就必须明确的传入n，比如`power(5, 3)`。

如上例所示，默认参数可以简化函数的调用。设置默认参数时，有几点要注意的地方：

- 必选参数在前，默认参数在后，否则Python的解释器会报错

- 如何设置默认参数

当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面，变化小的参数就可以作为默认参数。

使用默认参数有什么好处？ 最大的好处就是能降低调用函数的难度。

例：写一个小学生注册的函数，需要传入`name`和`gender`两个参数：

```python
def enroll(name, gender):
  print('name:', name)
  print('gender:', gender
```

这样，调用`enroll()`函数需要传入两个参数：

```python
>>> enroll('Ryan', 'M')
name: Ryan
gender: M
```

如果要继续传入年龄、城市等信息怎么办？ 这样会使得调用函数的复杂度大大增加。

可以把年龄和城市设为默认参数：

```
def enroll(name, gender, age = 6, city = 'SHENZHEN'):
  print('name:', name)
  print('gender:', gender)
  print('age:', age)
  print('city:', city)
```

这样，大多数学生注册的时候就不需要提供年龄与城市，只提供必须的两个参数：

```python
>>> enroll('Ryan', 'N')
name: Ryan
gender: M
age: 6
city: SHENZHEN
```

只有与默认参数不符的学生才需要提供额外的信息：

```python
enroll('Bob', 'M', 7)
enroll('Adam', 'M', city = 'BEIJING')
```

可见，默认参数降低了函数调用的难度，而一旦需要更复杂的调用时，又可以传递更多的参数来实现。无论是简单的调用还是复杂的调用，函数只需要定义一个。

有多个默认参数时，调用的时候，既可以按顺序提供默认参数，比如调用`enroll('Bob', 'M', 7)`，意思是，除了`name`、`gender`这两个参数外，最后一个参数应用在参数`age`上，`city`参数由于没有提供，仍然使用默认值。

当然，也可以不按顺序提供部分默认参数。当不按顺序提供默认参数时，需要把参数名加上。比如调用`enroll('Adam', 'M', city = 'BEIJING')`，意思是，'city'参数用传入的值，其它参数继续使用默认值。

默认参数很有用，但是如果使用不当，也会掉进坑里。默认参数最大的坑：

先定义一个函数，传入一个list，添加一个`END`再返回：

```python
def add_end(L=[]):
  L.append('END')
  return L
```

当你正常调用时，结果是正常的：

```python
>>> add_end([1, 2, 3])
[1, 2, 3, 'END']
>>> add_end(['x', 'y', 'z'])
['x', 'y', 'z', 'END']
```

当你使用默认参数调用时，结果也是正常的：

```python
>>> add_end()
['END']
```

但是，再次调用`add_end()`的时候，结果就不对了：

```python
>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
```

为什么呢？

因为Python函数在定义的时候，默认参数`L`的值就被计算出来了，即`[]`。因为默认参数`L`也是一个变量，它指向对象`[]`，每次调用该函数，如果改变了`L`的内容，则下次调用时，默认参数的内容就变了，不再是函数定义时的`[]`了。

> 定义默认参数要牢记的一点：默认参数必须指向不变对象！

修改上面的示例，可以用`None`这个不变对象来实现：

```python
def add_end(L=None):
    if L is None:
        L = []
    L.append('END')
    return L
```

现在，无论调用多少次，都不会出问题：

```python
>>> add_end()
['END']
>>> add_end()
['END']
```

为什么要设计`str`、`None`这样的不变对象呢？

因为不变对象一旦创建，对象内部的数据就不能被修改，这样就减少了由于修改数据导致的错误。此外，由于对象不变，多任务环境下同时读取对象不需要加锁，同时读一点问题都没有。我们在编写程序的时候，如果可以设计一个不变对象，那就尽量设计为不变对象。









