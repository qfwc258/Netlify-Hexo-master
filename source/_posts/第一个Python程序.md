title: 第一个Python程序
date: 2017-11-28 17:39:44
comments: true
tags: 
 - Python
categories: Python
----------

在正式编写第一个Python程序前，先复习一下什么是命令行模式和Python交互模式。

## 命令行模式

打开命令提示符窗口，就进入到了命令行模式

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171128171606.jpg)

<!-- more -->

## Python交互模式

在命令行模式下输入命令 `python`，会输出一堆文本，此时就进入到了Python交互模式。它的提示符是`>>>`

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171128171508.jpg)

在Python交互模式下输入`exit()`并回车，就会退出Python交互模式然后回到命令行模式

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171128171522.jpg)

也可以直接通过开始菜单选择`Python`菜单项，直接进入Python交互模式，但是输入'exit()'后窗口会直接关闭，不会回到命令行模式。

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171128171758.jpg)

**了解了如何启动和退出Python交互模式后，就可以正式开始编写Python代码了**

在Python交互模式的提示符`>>>`下，直接输入代码，按下回车，就可以得到执行结果

输入`1 + 2`

```python
>>> 1+2
3
>>>
```

任何有效的数学计算都可以算出来。

如果需要让Python打印出指定的文字，可以采用`print()`函数，然后把希望打印出来的文字用单引号或者双引号括起来，但是不能混用。
```Python
>>> print('hello,world')
hello,world
>>>
```

这种用单引号或双引号括起来的文本在程序中被称为字符串。

## 命令行模式和Python交互模式

请注意区分命令行模式和Python交互模式。

在命令行模式下，可以执行`python`进入Python交互式环境，也可以执行`python hello.py`运行一个`.py`文件。

执行一个`.py`文件只能在命令行模式执行。如果敲一个命令`python hello.py`，看到如下错误：

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171128173011.jpg)

错误提示`No such file or directory`说明这个`hello.py`在当前目录中不存在。必须先切换至`hello.py`所在目录下，才能正常执行

![](http://oih7sazbd.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171128173440.jpg)

此外，在命令行模式运行`.py`文件和在Python交互式环境下直接运行Python代码有所不同。Python交互式环境会把每一行Python代码的结果自动打印出来，但是，直接运行Python代码却不会。

例如：在Python交互环境下，输入：
```Python
>>> 100 + 200 + 300
600
>>>
```

直接可以看到结果`600`。

但是，写一个`calc.py`文件，内容如下：

```python
100 + 200 + 300
```

然后在命令模式下执行 `python calc.py `，会发现什么都无法输出。因为想要输出结果，必须自己用`print()`函数打印出来。

把`calc.py`改为：

```python
print(100+200+300)
```

然后在命令行模式下执行 `python calc.py`，就可以看到结果：
```python
python calc.py
600
```

最后，Python交互模式的代码是输入一行，执行一行，而命令行模式下直接运行.py文件是一次性执行该文件内的所有代码。可见，Python交互模式主要是为了调试Python代码用的，也便于初学者学习，它不是正式运行Python代码的环境！

## 小结
在Python交互式模式下，可以直接输入代码，然后执行，并立刻得到结果。

在命令行模式下，可以直接运行`.py`文件。

