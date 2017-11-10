title: 详解SpringMVC中Controller的方法中参数的工作原理[附带源码分析]
date: 2017-08-05
comments: true
tags:
 - SpringMVC
 - Controller
 - Java
categories: SpringMVC

----------
SpringMVC是目前主流的Web MVC框架之一。 

如果有同学对它不熟悉，那么请参考它的入门blog：[SpringMVC入门](https://blog.lyu3.com/springmvc%E5%85%A5%E9%97%A8/)

SpringMVC中Controller的方法参数可以是Integer，Double，自定义对象，ServletRequest，ServletResponse，ModelAndView等等，非常灵活。本文将分析SpringMVC是如何对这些参数进行处理的，使读者能够处理自定义的一些参数。

<!-- more -->

## 现象
本文使用的demo基于maven。我们先来看一看对应的现象。 

```java
@Controller
@RequestMapping(value = "/test")
public class TestController {
  
  @RequestMapping("/testRb")
  @ResponseBody
  public Employee testRb(@RequestBody Employee e) {
    return e;
  }
  
  @RequestMapping("/testCustomObj")
  @ResponseBody
  public Employee testCustomObj(Employee e) {
    return e;
  }
  
  @RequestMapping("/testCustomObjWithRp")
  @ResponseBody
  public Employee testCustomObjWithRp(@RequestParam Employee e) {
    return e;
  }
  
  @RequestMapping("/testDate")
  @ResponseBody
  public Date testDate(Date date) {
    return date;
  }
  
}
```

首先这是一个Controller，有4个方法。他们对应的参数分别是带有@RequestBody的自定义对象、自定义对象、带有@RequestParam的自定义对象、日期对象。

接下来我们一个一个方法进行访问看对应的现象是如何的。

首先第一个testRb：

![](http://images.cnitblog.com/i/411512/201405/151549179063987.jpg)

第二个testCustomObj：

![](http://images.cnitblog.com/i/411512/201405/151549409848203.jpg)

第三个testCustomObjWithRp：

![](http://images.cnitblog.com/i/411512/201405/151549564686102.jpg)

第四个testDate：

![](http://images.cnitblog.com/i/411512/201405/151550068591409.jpg)

为何返回的Employee对象会被自动解析为xml，请看楼主的另一篇博客：[SpringMVC关于json、xml自动转换的原理研究[附带源码分析]](https://blog.lyu3.com/springmvc%E5%85%B3%E4%BA%8Ejson%E3%80%81xml%E8%87%AA%E5%8A%A8%E8%BD%AC%E6%8D%A2%E7%9A%84%E5%8E%9F%E7%90%86%E7%A0%94%E7%A9%B6/)

为何Employee参数会被解析，带有@RequestParam的Employee参数不会被解析，甚至报错？

为何日期类型不能被解析？

SpringMVC到底是如何处理这些方法的参数的？

@RequestBody、@RequestParam这两个注解有什么区别？

带着这几个问题。我们开始进行分析。

## 源码分析
本文所分析的源码是Spring版本4.0.2

在分析源码之前，首先让我们来看下SpringMVC中两个重要的接口。

两个接口分别对应请求方法参数的处理、响应返回值的处理，分别是HandlerMethodArgumentResolver和HandlerMethodReturnValueHandler，这两个接口都是Spring3.1版本之后加入的。

![](http://images.cnitblog.com/i/411512/201405/151811284688188.jpg)
![](http://images.cnitblog.com/i/411512/201405/151811327507962.jpg)

SpringMVC处理请求大致是这样的：

首先被DispatcherServlet截获，DispatcherServlet通过handlerMapping获得HandlerExecutionChain，然后获得HandlerAdapter。

HandlerAdapter在内部对于每个请求，都会实例化一个ServletInvocableHandlerMethod进行处理，ServletInvocableHandlerMethod在进行处理的时候，会分两部分别对请求跟响应进行处理。

之后HandlerAdapter得到ModelAndView，然后做相应的处理。

本文将重点介绍ServletInvocableHandlerMethod对请求以及响应的处理。

![](http://images.cnitblog.com/i/411512/201405/161001448906593.jpg)

1 处理请求的时候，会根据ServletInvocableHandlerMethod的属性argumentResolvers（这个属性是它的父类InvocableHandlerMethod中定义的）进行处理，其中argumentResolvers属性是一个HandlerMethodArgumentResolverComposite类(这里使用了组合模式的一种变形)，这个类是实现了HandlerMethodArgumentResolver接口的类，里面有各种实现了HandlerMethodArgumentResolver的List集合。

![](http://images.cnitblog.com/i/411512/201405/152047402658258.jpg)

2 处理响应的时候，会根据ServletInvocableHandlerMethod的属性returnValueHandlers(自身属性)进行处理，returnValueHandlers属性是一个HandlerMethodReturnValueHandlerComposite类(这里使用了组合模式的一种变形)，这个类是实现了HandlerMethodReturnValueHandler接口的类，里面有各种实现了HandlerMethodReturnValueHandler的List集合。

![](http://images.cnitblog.com/i/411512/201405/152047089217350.jpg)

ServletInvocableHandlerMethod的returnValueHandlers和argumentResolvers这两个属性都是在ServletInvocableHandlerMethod进行实例化的时候被赋值的(使用RequestMappingHandlerAdapter的属性进行赋值)。

![](http://images.cnitblog.com/i/411512/201405/161034409068576.jpg)

RequestMappingHandlerAdapter的argumentResolvers和returnValueHandlers这两个属性是在RequestMappingHandlerAdapter进行实例化的时候被Spring容器注入的。

![](http://images.cnitblog.com/i/411512/201405/161038418281813.jpg)

其中默认的ArgumentResolvers:
![](http://images.cnitblog.com/i/411512/201405/161039256871346.jpg)

默认的returnValueHandlers：

![](http://images.cnitblog.com/i/411512/201405/161039592186357.jpg)

我们在[json、xml自动转换]()那篇文章中已经了解，使用@ResponseBody注解的话最终返回值会被RequestResponseBodyMethodProcessor这个HandlerMethodReturnValueHandler实现类处理。

我们通过源码发现，RequestResponseBodyMethodProcessor这个类其实同时实现了HandlerMethodReturnValueHandler和HandlerMethodArgumentResolver这两个接口。

![](http://images.cnitblog.com/i/411512/201405/151824306252641.jpg)

RequestResponseBodyMethodProcessor支持的请求类型是Controller方法参数中带有@RequestBody注解，支持的响应类型是Controller方法带有@ResponseBody注解。 
![](http://images.cnitblog.com/i/411512/201405/160114074066819.png)

RequestResponseBodyMethodProcessor响应的具体处理是使用消息转换器。

![](http://images.cnitblog.com/i/411512/201405/160117055008494.png)

处理请求的时候使用内部的readWithMessageConverters方法。

![](http://images.cnitblog.com/i/411512/201405/161007533591796.jpg)

然后会执行父类(AbstractMessageConverterMethodArgumentResolver)的readWithMessageConverters方法。

![](http://images.cnitblog.com/i/411512/201405/161012002035807.jpg)

下面来我们来看看常用的HandlerMethodArgumentResolver实现类(本文粗略讲下，有兴趣的读者可自行研究)。

1 RequestParamMethodArgumentResolver

 支持带有@RequestParam注解的参数或带有MultipartFile类型的参数

2 RequestParamMapMethodArgumentResolver

  支持带有@RequestParam注解的参数 && @RequestParam注解的属性value存在 && 参数类型是实现Map接口的属性

3 PathVariableMethodArgumentResolver

支持带有@PathVariable注解的参数 且如果参数实现了Map接口，@PathVariable注解需带有value属性

4 MatrixVariableMethodArgumentResolver

支持带有@MatrixVariable注解的参数 且如果参数实现了Map接口，@MatrixVariable注解需带有value属性 

5 RequestResponseBodyMethodProcessor

 本文已分析过

6 ServletRequestMethodArgumentResolver

 参数类型是实现或继承或是WebRequest、ServletRequest、MultipartRequest、HttpSession、Principal、Locale、TimeZone、InputStream、Reader、HttpMethod这些类。

（这就是为何我们在Controller中的方法里添加一个HttpServletRequest参数，Spring会为我们自动获得HttpServletRequest对象的原因）

7 ServletResponseMethodArgumentResolver

 参数类型是实现或继承或是ServletResponse、OutputStream、Writer这些类

8 RedirectAttributesMethodArgumentResolver

 参数是实现了RedirectAttributes接口的类

9 HttpEntityMethodProcessor

 参数类型是HttpEntity

从名字我们也看的出来， 以Resolver结尾的是实现了HandlerMethodArgumentResolver接口的类，以Processor结尾的是实现了HandlerMethodArgumentResolver和HandlerMethodReturnValueHandler的类。

 

下面来我们来看看常用的HandlerMethodReturnValueHandler实现类。

1 ModelAndViewMethodReturnValueHandler

返回值类型是ModelAndView或其子类

2 ModelMethodProcessor

返回值类型是Model或其子类

3. ViewMethodReturnValueHandler

返回值类型是View或其子类 

4 HttpHeadersReturnValueHandler

返回值类型是HttpHeaders或其子类  

5 ModelAttributeMethodProcessor

返回值有@ModelAttribute注解

6 ViewNameMethodReturnValueHandler

返回值是void或String

其余没讲过的读者可自行查看源码。

 

下面开始解释为何本文开头出现那些现象的原因：

  1 第一个方法testRb以及地址 http://localhost:8888/SpringMVCDemo/test/testRb?name=1&age=3

　　这个方法的参数使用了@RequestBody，之前已经分析过，被RequestResponseBodyMethodProcessor进行处理。之后根据http请求头部的contentType然后选择合适的消息转换器进行读取。

　　很明显，我们的消息转换器只有默认的那些跟部分json以及xml转换器，且传递的参数name=1&age=3，传递的头部中没有content-type，默认使用了application/octet-stream，因此触发了HttpMediaTypeNotSupportedException异常

　　解放方案： 我们将传递数据改成json，同时http请求的Content-Type改成application/json即可。

![](http://images.cnitblog.com/i/411512/201405/171148152966518.png)

![](http://images.cnitblog.com/i/411512/201405/171148496253471.png)
![](http://images.cnitblog.com/i/411512/201405/171148581408407.png)

完美解决。
  
  2 testCustomObj方法以及地址 http://localhost:8888/SpringMVCDemo/test/testCustomObj?name=1&age=3

这个请求会找到ServletModelAttributeMethodProcessor这个resolver。默认的resolver中有两个ServletModelAttributeMethodProcessor，只不过实例化的时候属性annotationNotRequired一个为true，1个为false。这个ServletModelAttributeMethodProcessor处理参数支持@ModelAttribute注解，annotationNotRequired属性为true的话，参数不是简单类型就通过，因此选择了ServletModelAttributeMethodProcessor，最终通过DataBinder实例化Employee对象，并写入对应的属性。

 3 testCustomObjWithRp方法以及地址 http://localhost:8888/SpringMVCDemo/test/testCustomObjWithRp?name=1&age=3

这个请求会找到RequestParamMethodArgumentResolver(使用了@RequestParam注解)。RequestParamMethodArgumentResolver在处理参数的时候使用request.getParameter(参数名)即request.getParameter("e")得到，很明显我们的参数传的是name=1&age=3。因此得到null，RequestParamMethodArgumentResolver处理missing value会触发MissingServletRequestParameterException异常。 ［粗略讲下，有兴趣的读者请自行查看源码］

  解决方案：去掉@RequestParam注解，让ServletModelAttributeMethodProcessor来处理。

 4 testDate方法以及地址 http://localhost:8888/SpringMVCDemo/test/testDate?date=2014-05-15

这个请求会找到RequestParamMethodArgumentResolver。因为这个方法与第二个方法一样，有两个RequestParamMethodArgumentResolver，属性useDefaultResolution不同。RequestParamMethodArgumentResolver支持简单类型，ServletModelAttributeMethodProcessor是支持非简单类型。最终步骤跟第三个方法一样，我们的参数名是date，于是通过request.getParameter("date")找到date字符串(这里参数名如果不是date，那么最终页面是空白的，因为没有@RequestParam注解，参数不是必须的，RequestParamMethodArgumentResolver处理null值返回null)。最后通过DataBinder找到合适的属性编辑器进行类型转换。最终找到java.util.Date对象的构造函数 public Date(String s)，由于我们传递的格式不是标准的UTC时间格式，因此最终触发了IllegalArgumentException异常。

解决方案：

1 传递参数的格式修改成标准的UTC时间格式：http://localhost:8888/SpringMVCDemo/test/testDate?date=Sat, 17 May 2014 16:30:00 GMT

![](http://images.cnitblog.com/i/411512/201405/171551382342264.png)

2 在Controller中加入自定义属性编辑器。

 ```java
@InitBinder
public void initBinder(WebDataBinder binder) {
  SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
  binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
}
```

 这个@InitBinder注解在实例化ServletInvocableHandlerMethod的时候被注入到WebDataBinderFactory中的，而WebDataBinderFactory是ServletInvocableHandlerMethod的一个属性。在RequestMappingHandlerAdapter源码的803行getDataBinderFactory就是得到的WebDataBinderFactory。

 ![](http://images.cnitblog.com/i/411512/201405/171742265315423.png)

 之后RequestParamMethodArgumentResolver通过WebDataBinderFactory创建的WebDataBinder里的自定义属性编辑器找到合适的属性编辑器(我们自定义的属性编辑器是用CustomDateEditor处理Date对象，而testDate的参数刚好是Date)，最终CustomDateEditor把这个String对象转换成Date对象。

 ## 编写自定义的HandlerMethodArgumentResolver
 通过前面的分析，我们明白了SpringMVC处理Controller中的方法的参数流程。

现在，如果方法中有两个参数，且都是自定义类参数，那该如何处理呢？

很明显，要处理这个只能自己实现一个实现HandlerMethodArgumentResolver的类。

先定义1个注解FormObj：

```java
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface FormObj {
    //参数别名
    String value() default "";
    //是否展示, 默认展示
    boolean show() default true;
}
```

然后是HandlerMethodArgumentResolver：

```java
public class FormObjArgumentResolver implements HandlerMethodArgumentResolver {
  
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(FormObj.class);
    }
    
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        FormObj formObj = parameter.getParameterAnnotation(FormObj.class);

        String alias = getAlias(formObj, parameter);

        //拿到obj, 先从ModelAndViewContainer中拿，若没有则new1个参数类型的实例
        Object obj = (mavContainer.containsAttribute(alias)) ?
                mavContainer.getModel().get(alias) : createAttribute(alias, parameter, binderFactory, webRequest);


        //获得WebDataBinder，这里的具体WebDataBinder是ExtendedServletRequestDataBinder
        WebDataBinder binder = binderFactory.createBinder(webRequest, obj, alias);

        Object target = binder.getTarget();

        if(target != null) {
            //绑定参数
            bindParameters(webRequest, binder, alias);
            //JSR303 验证
            validateIfApplicable(binder, parameter);
            if (binder.getBindingResult().hasErrors()) {
                if (isBindExceptionRequired(binder, parameter)) {
                    throw new BindException(binder.getBindingResult());
                }
            }
        }

        if(formObj.show()) {
            mavContainer.addAttribute(alias, target);
        }

        return target;
    }
    
    
    private Object createAttribute(String alias, MethodParameter parameter, WebDataBinderFactory binderFactory, NativeWebRequest webRequest) {
        return BeanUtils.instantiateClass(parameter.getParameterType());
    }
    
    private void bindParameters(NativeWebRequest request, WebDataBinder binder, String alias) {
        ServletRequest servletRequest = request.getNativeRequest(ServletRequest.class);

        MockHttpServletRequest newRequest = new MockHttpServletRequest();

        Enumeration<String> enu = servletRequest.getParameterNames();
        while(enu.hasMoreElements()) {
            String paramName = enu.nextElement();
            if(paramName.startsWith(alias)) {
                newRequest.setParameter(paramName.substring(alias.length()+1), request.getParameter(paramName));
            }
        }
        ((ExtendedServletRequestDataBinder)binder).bind(newRequest);
    }
    
    protected void validateIfApplicable(WebDataBinder binder, MethodParameter parameter) {
        Annotation[] annotations = parameter.getParameterAnnotations();
        for (Annotation annot : annotations) {
            if (annot.annotationType().getSimpleName().startsWith("Valid")) {
                Object hints = AnnotationUtils.getValue(annot);
                binder.validate(hints instanceof Object[] ? (Object[]) hints : new Object[] {hints});
                break;
            }
        }
    }
    
    protected boolean isBindExceptionRequired(WebDataBinder binder, MethodParameter parameter) {
        int i = parameter.getParameterIndex();
        Class<?>[] paramTypes = parameter.getMethod().getParameterTypes();
        boolean hasBindingResult = (paramTypes.length > (i + 1) && Errors.class.isAssignableFrom(paramTypes[i + 1]));

        return !hasBindingResult;
    }
    
    private String getAlias(FormObj formObj, MethodParameter parameter) {
        //得到FormObj的属性value，也就是对象参数的简称
        String alias = formObj.value();
        if(alias == null || StringUtils.isBlank(alias)) {
            //如果简称为空，取对象简称的首字母小写开头
            String simpleName = parameter.getParameterType().getSimpleName();
            alias = simpleName.substring(0, 1).toLowerCase() + simpleName.substring(1);
        }
        return alias;
    }

  
}
```
对应Controller：
```java
@Controller
@RequestMapping(value = "/foc")
public class FormObjController {
  
    @RequestMapping("/test1")
    public String test1(@FormObj Dept dept, @FormObj Employee emp) {
        return "index";
    }
    
    @RequestMapping("/test2")
    public String test2(@FormObj("d") Dept dept, @FormObj("e") Employee emp) {
        return "index";
    }
    
    @RequestMapping("/test3")
    public String test3(@FormObj(value = "d", show = false) Dept dept, @FormObj("e") Employee emp) {
        return "index";
    }
  
}
```

结果如下：

![](http://images.cnitblog.com/i/411512/201406/250138599391765.png)

![](http://images.cnitblog.com/i/411512/201406/250140001743868.png)

![](http://images.cnitblog.com/i/411512/201406/250140391273568.png)

## 总结
写了这么多，主要还是巩固一下自己对SpringMVC对请求及响应的处理做一个细节的总结吧，不知道大家有没有清楚这个过程。

想熟悉这部分内容最主要的还是要熟悉HandlerMethodArgumentResolver和HandlerMethodReturnValueHandler这两个接口以及属性编辑器、数据绑定机制。
    
