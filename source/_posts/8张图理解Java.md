title: 8 张图理解 Java
date: 2017/10/15
comments: true
tags: 
 - Java
categories: 
 - 后端技术
----------

一图胜千言，图片均来源于Program Creek [](https://www.programcreek.com/)
<!-- more -->

## 1、字符串不变性

下面这张图展示了这段代码做了什么

```java
String s = "abcd";
s = s.concat("ef");
```
![](http://mmsns.qpic.cn/mmsns/eZzl4LXykQxarEibIxzaFfxibZznAkQh1EJoLJU1Y6ibyIWTcfOs4ictNA/0?wx_lazy=1)

## 2、equals()方法、hashCode()方法的区别

HashCode被设计用来提高性能。equals()方法与hashCode()方法的区别在于：

如果两个对象相等(equal)，那么他们一定有相同的哈希值。

如果两个对象的哈希值相同，但他们未必相等(equal)。

![](http://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQxarEibIxzaFfxibZznAkQh1ELKypAia7Xngm8Rrqke6h3BWBRSLTOTgNQCzRRsoEMtZEq8qnfqO7X4A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

## 3、Java异常类的层次结构

图中红色部分为受检查异常。它们必须被捕获，或者在函数中声明为抛出该异常。

![](http://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQxarEibIxzaFfxibZznAkQh1EKr8DE7Btv4Wn9SEt1vBzOqGn3H6ebMh1ib23DgR5LYnZkgcAOGzDXcA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)
## 4、集合类的层次结构

注意Collections和Collection的区别。（Collections包含有各种有关集合操作的静态多态方法）

![](http://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQxarEibIxzaFfxibZznAkQh1EH6icXCT40kUDiaYPvhybrHVOEhqsXiayW4Hgz57kd72kc8Hvu7bBj13LQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

## 5、Java同步

Java同步机制可通过类比建筑物来阐明。

![](http://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQxarEibIxzaFfxibZznAkQh1En7buV49RmtK9XxdcQvc8ILxbw2LicSDfsmGCNjd7yxz62Z4Va4sUDDA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

## 6、别名

别名意味着有多个变量指向同一可被更新的内存块，这些别名分别是不同的对象类型。

![](http://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQxarEibIxzaFfxibZznAkQh1Es03MyWlC2pjyFohQAu13gsOldxZWsKDHnSr0assh9cc4tr3rhibbZ2Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

## 7、堆和栈

图解表明了方法和对象在运行时内存中的位置。

![](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQxarEibIxzaFfxibZznAkQh1Ek71knIAFeg7ngqDvB62iargv6M75asiaTx5Jmj9ns6svKMRuOxAVEHPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

## 8、Java虚拟机运行时数据区域

图解展示了整个虚拟机运行时数据区域的情况。

![](http://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQxarEibIxzaFfxibZznAkQh1EcKIFlcamNlkDNgvKyyUwCKuzRF0ibcVyaDfU7JnxzLVfIdL5jlaVP1w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)