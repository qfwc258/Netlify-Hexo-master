title: SpringMVC类型转换、数据绑定详解[附带源码分析]
date: 2017-08-07
comments: true
tag:
 - SpringMVC
 - 数据绑定
 - Java
categories: SpringMVC

----------
SpringMVC是目前主流的Web MVC框架之一。 

如果有同学对它不熟悉，那么请参考它的入门blog：[SpringMVC入门](https://blog.lyu3.com/springmvc%E5%85%A5%E9%97%A8/)
<!-- more -->

```java
public String method(Integer num, Date birth) {
  ...
}
```
Http请求传递的数据都是字符串String类型的，上面这个方法在Controller中定义，如果该方法对应的地址接收到到浏览器的请求的话，并且请求中含有num和birth参数，那么num会被自动转换成Integer对象；birth会被自动转为Date对象(Date转换需要配置属性编辑器)。

![](http://images.cnitblog.com/i/411512/201405/311217211973272.jpg)

本文将分析这一原理，解释SpringMVC是如何实现数据类型的转换。

## 属性编辑器介绍

在讲解核心内容之前，我们先来了解一下Java中定义的属性编辑器。

sun设计属性编辑器主要是为IDE服务的，让IDE能够以可视化的方式设置JavaBean的属性。

PropertyEditor是属性编辑器的接口。

![](http://images.cnitblog.com/i/411512/201405/311233239161256.png)

我们使用属性编辑器一般都是将String对象转换成我们需要的java对象而使用的。

有个方法setAsText很重要。 比如String对象"1"要使用属性编辑器转换成Integer对象，通过setAsText里Integer.parseInt(text)得到Integer对象，然后将Integer对象保存到属性中。

它的基本实现类是PropertyEditorSupport，一般我们要编写自定义的属性编辑器只需要继承这个类即可。

Spring中有很多自定义的属性编辑器，都在spring-beans jar包下的org.springframework.beans.propertyeditors包里。

![](http://images.cnitblog.com/i/411512/201405/311355301192607.png)

CustomBooleanEditor继承PropertyEditorSupport并重写setAsText方法。

![](http://images.cnitblog.com/i/411512/201405/311433035567401.png)

## 重要接口和类介绍
   
刚刚分析了sun设计的属性编辑器。 下面我们来看下Spring对这方面的设计。
   
1 PropertyEditorRegistry接口
   
封装方法来给JavaBean注册对应的属性编辑器。

![](http://images.cnitblog.com/i/411512/201405/311558041343681.png)

2 PropertyEditorRegistrySupport：PropertyEditorRegistry接口的基础实现类

![](http://images.cnitblog.com/i/411512/201405/311614164789055.png)

PropertyEditorRegistrySupport类有个createDefaultEditors方法，会创建默认的属性编辑器。

![](http://images.cnitblog.com/i/411512/201405/311624538389040.png)

![](http://images.cnitblog.com/i/411512/201405/311625049787402.png)

3 TypeConverter接口
 
 类型转换接口。 通过该接口，可以将value转换为requiredType类型的对象。

![](http://images.cnitblog.com/i/411512/201405/311635225259071.png)

4 TypeConverterSupport：TypeConverter基础实现类，并继承了PropertyEditorRegistrySupport　　

　　有个属性typeConverterDelegate，类型为TypeConverterDelegate，TypeConverterSupport将类型转换委托给typeConverterDelegate操作。

5 TypeConverterDelegate

　　类型转换委托类。具体的类型转换操作由此类完成。

6 SimpleTypeConverter

　　TypeConverterSupport的子类，使用了PropertyEditorRegistrySupport(父类TypeConverterSupport的父类PropertyEditorRegistrySupport)中定义的默认属性编辑器。

7 PropertyAccessor接口

　　对类中属性操作的接口。

8 BeanWrapper接口

　　继承ConfigurablePropertyAccessor(继承PropertyAccessor、PropertyEditorRegistry、TypeConverter接口)接口的操作Spring中JavaBean的核心接口。

9 BeanWrapperImpl类

　　BeanWrapper接口的默认实现类，TypeConverterSupport是它的父类，可以进行类型转换，可以进行属性设置。

10 DataBinder类

　　实现PropertyEditorRegistry、TypeConverter的类。支持类型转换，参数验证，数据绑定等功能。

　　有个属性SimpleTypeConverter，用来进行类型转换操作。

11 WebDataBinder

　　DataBinder的子类，主要是针对Web请求的数据绑定。

## 部分类和接口测试

由于BeanWrapper支持类型转换，属性设置。以BeanWrapper接口为例，做几个测试，让读者对它们有更清晰的认识：

以TestModel这个JavaBean为例，属性：

```java
  private int age;
  private Date birth;
  private String name;
  private boolean good;
  private long times;
```

测试方法1：

```java
TestModel tm = new TestModel();
BeanWrapper bw = new BeanWrapperImpl(tm);
bw.setPropertyValue("good", "on");
//bw.setPropertyValue("good", "1");
//bw.setPropertyValue("good", "true");
//bw.setPropertyValue("good", "yes");
System.out.println(tm);
```

good是boolean属性，使用BeanWrapperImpl设置属性的时候，内部会使用类型转换(父类TypeConverterSupport提供)，将String类型转换为boolean，CustomBooleanEditor对于String值是on，1，true，yes都会转换为true，本文介绍PropertyEditorRegistrySupport的时候说明过，CustomBooleanEditor属于默认的属性编辑器。

测试方法2：

```java
TestModel tm = new TestModel();
BeanWrapperImpl bw = new BeanWrapperImpl(false);
bw.setWrappedInstance(tm);
bw.setPropertyValue("good", "1");
System.out.println(tm);
```

不使用默认的属性编辑器进行类型转换。 很明显，这段代码报错了，没有找到合适的属性编辑，String类型不能作为boolean类型的值。

错误信息：Failed to convert property value of type 'java.lang.String' to required type 'boolean' for property 'good'; 

测试方法3：

```java
TestModel tm = new TestModel();
BeanWrapper bw = new BeanWrapperImpl(tm);
bw.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
bw.setPropertyValue("birth", "1990-01-01");
System.out.println(tm);
```

默认属性编辑器中并没有日期类型的属性编辑器，我们注册一个Spring提供的CustomDateEditor属性编辑器，对应Date对象，并且为空。有了CustomDateEditor，设置birth的时候会通过类型转换自动转化成Date对象。

关于其他属性的设置，读者自行测试吧。

## 源码分析
本文所使用的Spring版本是4.0.2

在分析RequestParamMethodArgumentResolver处理请求参数之前，我们简单回顾一下SpringMVC是如何对http请求进行处理的。

HandlerAdapter会对每个请求实例化一个ServletInvocableHandlerMethod对象进行处理，我们仅看下WebDataBinderFactory的构造过程。

WebDataBinderFactory接口是一个创建WebDataBinder的工厂接口。

![](http://images.cnitblog.com/i/411512/201406/011331475252863.png)

![](http://images.cnitblog.com/i/411512/201406/011332128695987.png)

![](http://images.cnitblog.com/i/411512/201406/011332254476578.png)

以如下方法为例：

```java
public ModelAndView test(boolean b, ModelAndView view) {
  view.setViewName("test/test");
  if(b) {
      view.addObject("attr", "b is true");
  } else {
      view.addObject("attr", "b is false");
  }
  return view;
}
```

楼主在另外一篇博客[详解SpringMVC中Controller的方法中参数的工作原理](https://blog.lyu3.com/%E8%AF%A6%E8%A7%A3springmvc%E4%B8%ADcontroller%E7%9A%84%E6%96%B9%E6%B3%95%E4%B8%AD%E5%8F%82%E6%95%B0%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/)中已经分析，boolean类型的参数会被RequestParamMethodArgumentResolver这个HandlerMethodArgumentResolver处理。

下面我们进入RequestParamMethodArgumentResolver看看是如何处理的。

RequestParamMethodArgumentResolver的resolveArgument方法是由它的父类AbstractNamedValueMethodArgumentResolver中定义的：

![](http://images.cnitblog.com/i/411512/201406/011415110097085.png)

ServletRequestDataBinderFactory创建ExtendedServletRequestDataBinder。

![](http://images.cnitblog.com/i/411512/201406/011449299787124.png)

ExtendedServletRequestDataBinder属于DataBinder的子类。

![](http://images.cnitblog.com/i/411512/201406/011419117442906.png)

我们在介绍重要接口的时候说过DataBinder进行类型转换的时候内部会使用SimpleTypeConverter进行数据转换。

下面看看测试：

![](http://images.cnitblog.com/i/411512/201406/011516384471985.png)
![](http://images.cnitblog.com/i/411512/201406/011516446195666.png)
![](http://images.cnitblog.com/i/411512/201406/011517379311511.png)
![](http://images.cnitblog.com/i/411512/201406/011517318537801.png)
![](http://images.cnitblog.com/i/411512/201406/011517447755821.png)

PS：以上例子boolean类型改成Boolean类型的话，不传参数的话b就是null，我们解释默认属性编辑器的时候Boolean类型的参数是允许空的。但是boolean类型不传参数的话，默认会是false，而不会抛出异常。 原因就是resolveArgument方法中handleNullValue处理null值，spring进行了特殊的处理，如果参数类型是boolean的话，取false。 读者可以试试。

再看看个例子：

```java
public ModelAndView testObj(Employee e, ModelAndView view) {
  view.setViewName("test/test");
  view.addObject("attr", e.toString());
  return view;
}
```

该方法会被ServletModelAttributeMethodProcessorr这个HandlerMethodArgumentResolver处理。

ServletModelAttributeMethodProcessorr的resolveArgument方法是由它的父类ModelAttributeMethodProcessor中定义的：

![](http://images.cnitblog.com/i/411512/201406/011657358841538.png)

这里WebDataBinder方法bind中会使用BeanWrapper构造对象，然后设置对应的属性。BeanWrapper本文已介绍过。

![](http://images.cnitblog.com/i/411512/201406/011714058534629.png)

## 编写自定义的属性编辑器
Spring提供的编辑器肯定不会满足我们日常开发的功能，因此开发自定义的属性编辑器也是很有必要的。

下面我们就写1个自定义的属性编辑器。

```java
public class CustomDeptEditor extends PropertyEditorSupport {
  
  @Override
  public void setAsText(String text) throws IllegalArgumentException { 
    if(text.indexOf(",") > 0) {
        Dept dept = new Dept();
        String[] arr = text.split(",");
        dept.setId(Integer.parseInt(arr[0]));
        dept.setName(arr[1]);
        setValue(dept);
    } else {
        throw new IllegalArgumentException("dept param is error");
    }
  }
  
}
```

SpringMVC中使用自定义的属性编辑器有3种方法：

1 Controller方法中添加@InitBinder注解的方法

```java
@InitBinder
public void initBinder(WebDataBinder binder) { 
  binder.registerCustomEditor(Dept.class, new CustomDeptEditor());  
}
```

2 实现WebBindingInitializer接口

```java
public class MyWebBindingInitializer implements WebBindingInitializer {
  
  @Override
  public void initBinder(WebDataBinder binder, WebRequest request) { 
    binder.registerCustomEditor(Dept.class, new CustomDeptEditor());  
  }
  
}
```

之前分析源码的时候，HandlerAdapter构造WebDataBinderFactory的时候，会传递HandlerAdapter的属性webBindingInitializer。

因此，我们在配置文件中构造RequestMappingHandlerAdapter的时候传入参数webBindingInitializer。

3 @ControllerAdvice注解

```java
@ControllerAdvice
public class InitBinderControllerAdvice {
  
  @InitBinder
  public void initBinder(WebDataBinder binder) { 
    binder.registerCustomEditor(Dept.class, new CustomDeptEditor());  
  }
  
}
```

加上ControllerAdvice别忘记配置文件component-scan需要扫描到这个类。

最终结果：

![](http://images.cnitblog.com/i/411512/201406/011824261816261.png)

## 总结 
分析了Spring的数据转换功能，并解释这个神奇的转换功能是如何实现的，之后编写了自定义的属性编辑器。

大致讲解了下Spring类型转换中重要的类及接口。