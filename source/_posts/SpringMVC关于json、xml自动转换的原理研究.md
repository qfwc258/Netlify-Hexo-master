title: SpringMVC关于json、xml自动转换的原理研究[附带源码分析]
date: 2017-08-06
comments: true
tags:
 - SpringMVC
 - JSON
 - XML
 - Java
categories: 后端技术
---

SpringMVC是目前主流的Web MVC框架之一。 

如果有同学对它不熟悉，那么请参考它的入门blog：[SpringMVC入门](https://blog.lyu3.com/springmvc%E5%85%A5%E9%97%A8/)
<!-- more -->
## 现象

本文使用的demo基于maven，是根据入门blog的例子继续写下去的。

我们先来看一看对应的现象。 我们这里的配置文件 *-dispatcher.xml中的关键配置如下(其他常规的配置文件不在讲解，可参考本文一开始提到的入门blog)：

(视图配置省略)

```xml
<mvc:resources location="/static/" mapping="/static/**"/>
<mvc:annotation-driven/>
<context:component-scan base-package="org.format.demo.controller"/>
```

pom中需要有以下依赖(Spring依赖及其他依赖不显示)：

```xml
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-asl</artifactId>
  <version>1.9.13</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>1.9.13</version>
</dependency>
```

这个依赖是json序列化的依赖。

ok。我们在Controller中添加一个method：

```java
@RequestMapping("/xmlOrJson")
@ResponseBody
public Map<String, Object> xmlOrJson() {
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("list", employeeService.list());
    return map;
}
```

直接访问地址：

![](http://images.cnitblog.com/i/411512/201405/101449596675807.png)

我们看到，短短几行配置。使用@ResponseBody注解之后，Controller返回的对象 自动被转换成对应的json数据，在这里不得不感叹SpringMVC的强大。

我们好像也没看到具体的配置，唯一看到的就是*-dispatcher.xml中的一句配置：<mvc:annotation-driven/>。其实就是这个配置，导致了java对象自动转换成json对象的现象。

那么spring到底是如何实现java对象到json对象的自动转换的呢？ 为什么转换成了json数据，如果想转换成xml数据，那该怎么办？

## 源码分析

本文使用的spring版本是4.0.2。  

在讲解<mvc:annotation-driven/>这个配置之前，我们先了解下Spring的消息转换机制。@ResponseBody这个注解就是使用消息转换机制，最终通过json的转换器转换成json数据的。

HttpMessageConverter接口就是Spring提供的http消息转换接口。有关这方面的知识大家可以参考"参考资料"中的第二条链接，里面讲的很清楚。

![](http://images.cnitblog.com/i/411512/201405/101510002604230.png)

下面开始分析<mvc:annotation-driven/>这句配置:

这句代码在spring中的解析类是：

![](http://images.cnitblog.com/i/411512/201405/101606162131470.png)

在AnnotationDrivenBeanDefinitionParser源码的152行parse方法中：

分别实例化了RequestMappingHandlerMapping，ConfigurableWebBindingInitializer，RequestMappingHandlerAdapter等诸多类。

其中RequestMappingHandlerMapping和RequestMappingHandlerAdapter这两个类比较重要。

RequestMappingHandlerMapping处理请求映射的，处理@RequestMapping跟请求地址之间的关系。

RequestMappingHandlerAdapter是请求处理的适配器，也就是请求之后处理具体逻辑的执行，关系到哪个类的哪个方法以及转换器等工作，这个类是我们讲的重点，其中它的属性messageConverters是本文要讲的重点。

![](http://images.cnitblog.com/i/411512/201405/101611179016436.png)

私有方法:getMessageConverters

![](http://images.cnitblog.com/i/411512/201405/101630232136603.png)

从代码中我们可以，RequestMappingHandlerAdapter设置messageConverters的逻辑：

1.如果<mvc:annotation-driven>节点有子节点message-converters，那么它的转换器属性messageConverters也由这些子节点组成。

message-converters的子节点配置如下：

```xml
<mvc:annotation-driven>
  <mvc:message-converters>
    <bean class="org.example.MyHttpMessageConverter"/>
    <bean class="org.example.MyOtherHttpMessageConverter"/>
  </mvc:message-converters>
</mvc:annotation-driven>
```

2.message-converters子节点不存在或它的属性register-defaults为true的话，加入其他的转换器：ByteArrayHttpMessageConverter、StringHttpMessageConverter、ResourceHttpMessageConverter等。

我们看到这么一段：

![](http://images.cnitblog.com/i/411512/201405/101640298384297.png)

这些boolean属性是哪里来的呢，它们是AnnotationDrivenBeanDefinitionParser的静态变量。

![](http://images.cnitblog.com/i/411512/201405/101641297132356.png)

其中ClassUtils中的isPresent方法如下：

![](http://images.cnitblog.com/i/411512/201405/101643277139672.png)

看到这里，读者应该明白了为什么本文一开始在pom文件中需要加入对应的jackson依赖，为了让json转换器jackson成为默认转换器之一。

<mvc:annotation-driven>的作用读者也明白了。

下面我们看如何通过消息转换器将java对象进行转换的。

 

RequestMappingHandlerAdapter在进行handle的时候，会委托给HandlerMethod（具体由子类ServletInvocableHandlerMethod处理）的invokeAndHandle方法进行处理，这个方法又转接给HandlerMethodReturnValueHandlerComposite处理。

HandlerMethodReturnValueHandlerComposite维护了一个HandlerMethodReturnValueHandler列表。HandlerMethodReturnValueHandler是一个对返回值进行处理的策略接口，这个接口非常重要。关于这个接口的细节，请参考楼主的另外一篇博客：[详解SpringMVC中Controller的方法中参数的工作原理[附带源码分析]](https://blog.lyu3.com/%E8%AF%A6%E8%A7%A3springmvc%E4%B8%ADcontroller%E7%9A%84%E6%96%B9%E6%B3%95%E4%B8%AD%E5%8F%82%E6%95%B0%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/)。然后找到对应的HandlerMethodReturnValueHandler对结果值进行处理。

最终找到RequestResponseBodyMethodProcessor这个Handler（由于使用了@ResponseBody注解）。

RequestResponseBodyMethodProcessor的supportsReturnType方法：

![](http://images.cnitblog.com/i/411512/201405/101803027605809.png)

然后使用handleReturnValue方法进行处理：

![](http://images.cnitblog.com/i/411512/201405/101803105889900.png)

我们看到，这里使用了转换器。　　

具体的转换方法：

![](http://images.cnitblog.com/i/411512/201405/101809037135949.png)

![](http://images.cnitblog.com/i/411512/201405/102031439173571.png)

至于为何是请求头部的Accept数据，读者可以进去debug这个getAcceptableMediaTypes方法看看。 我就不罗嗦了～～～

 ok。至此，我们走遍了所有的流程。

 

现在，回过头来看。为什么一开始的demo输出了json数据？

我们来分析吧。

 

由于我们只配置了<mvc:annotation-driven>，因此使用spring默认的那些转换器。

![](http://images.cnitblog.com/i/411512/201405/101816581047144.png)

很明显，我们看到了2个xml和1个json转换器。 要看能不能转换，得看HttpMessageConverter接口的public boolean canWrite(Class<?> clazz, MediaType mediaType)方法是否返回true来决定的。

我们先分析SourceHttpMessageConverter：

它的canWrite方法被父类AbstractHttpMessageConverter重写了。

![](http://images.cnitblog.com/i/411512/201405/101830573234896.png)

![](http://images.cnitblog.com/i/411512/201405/101832284176592.png)

![](http://images.cnitblog.com/i/411512/201405/101832352929525.png)

发现SUPPORTED_CLASSES中没有Map类(本文demo返回的是Map类)，因此不支持。

下面看Jaxb2RootElementHttpMessageConverter：

这个类直接重写了canWrite方法。

![](http://images.cnitblog.com/i/411512/201405/101838053851073.png)

需要有XmlRootElement注解。 很明显，Map类当然没有。

最终MappingJackson2HttpMessageConverter匹配，进行json转换。（为何匹配，请读者自行查看源码）

## 实例讲解

 我们分析了转换器的转换过程之后，下面就通过实例来验证我们的结论吧。

首先，我们先把xml转换器实现。

之前已经分析，默认的转换器中是支持xml的。下面我们加上注解试试吧。

由于Map是jdk源码中的部分，因此我们用Employee来做demo。

因此，Controller加上一个方法：

```java
@RequestMapping("/xmlOrJsonSimple")
@ResponseBody
public Employee xmlOrJsonSimple() {
    return employeeService.getById(1);
}
```

实体中加上@XmlRootElement注解

![](http://images.cnitblog.com/i/411512/201405/101903141989122.png)

结果如下：

![](http://images.cnitblog.com/i/411512/201405/101904598389030.png)

我们发现，解析成了xml。

这里为什么解析成xml，而不解析成json呢？

 

之前分析过，消息转换器是根据class和mediaType决定的。

我们使用firebug看到：

![](http://images.cnitblog.com/i/411512/201405/102222464019898.png)

我们发现Accept有xml，没有json。因此解析成xml了。

我们再来验证，同一地址，HTTP头部不同Accept。看是否正确。

```javascript
$.ajax({
    url: "${request.contextPath}/employee/xmlOrJsonSimple",
    success: function(res) {
        console.log(res);
    },
    headers: {
        "Accept": "application/xml"
    }
});
```
![](http://images.cnitblog.com/i/411512/201405/102231412601968.png)

```javascript
$.ajax({
    url: "${request.contextPath}/employee/xmlOrJsonSimple",
    success: function(res) {
        console.log(res);
    },
    headers: {
        "Accept": "application/json"
    }
});
```
![](http://images.cnitblog.com/i/411512/201405/102231514794990.png)

验证成功。

## 关于配置

如果不想使用<mvc:annotation-driven/>中默认的RequestMappingHandlerAdapter的话，我们可以在重新定义这个bean，spring会覆盖掉默认的RequestMappingHandlerAdapter。

为何会覆盖，请参考楼主的另外一篇博客：[Spring中Ordered接口简介](https://blog.lyu3.com/Spring中Ordered接口简介/)

```xml
bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
  <property name="messageConverters">
    <list>
      <bean class="org.springframework.http.converter.ByteArrayHttpMessageConverter"/>
      <bean class="org.springframework.http.converter.StringHttpMessageConverter"/>
      <bean class="org.springframework.http.converter.ResourceHttpMessageConverter"/>
    </list>
  </property>
</bean>
```

或者如果只想换messageConverters的话。

```xml
mvc:annotation-driven>
  <mvc:message-converters>
    <bean class="org.example.MyHttpMessageConverter"/>
    <bean class="org.example.MyOtherHttpMessageConverter"/>
  </mvc:message-converters>
</mvc:annotation-driven>
```

如果还想用其他converters的话。

![](http://images.cnitblog.com/i/411512/201405/102311480731629.png)

以上是spring-mvc jar包中的converters。

这里我们使用转换xml的MarshallingHttpMessageConverter。

这个converter里面使用了marshaller进行转换

![](http://images.cnitblog.com/i/411512/201405/102313161827280.png)

我们这里使用XStreamMarshaller。　　

![](http://images.cnitblog.com/i/411512/201405/102319292603758.png)

![](http://images.cnitblog.com/i/411512/201405/102319412294581.png)

json没有转换器，返回406.

至于xml格式的问题，大家自行解决吧。 这里用的是XStream～。

使用这种方式，pom别忘记了加入xstream的依赖：

```xml
dependency>
  <groupId>com.thoughtworks.xstream</groupId>
  <artifactId>xstream</artifactId>
  <version>1.4.7</version>
</dependency>
```

## 总结
写了这么多，可能读者觉得有点罗嗦。 毕竟这也是自己的一些心得，希望都能说出来与读者共享。

刚接触SpringMVC的时候，发现这种自动转换机制很牛逼，但是一直没有研究它的原理，目前，算是了了一个小小心愿吧，SpringMVC还有很多内容，以后自己研究其他内容的时候还会与大家一起共享的。