title: SpringMVC重定向视图RedirectView小分析
date: 2017-08-10
comments: true
tag:
 - SpringMVC
 - RedirectView
 - Java
categories: SpringMVC

----
## 前言
SpringMVC是目前主流的Web MVC框架之一。 

如果有同学对它不熟悉，那么请参考它的入门blog：[SpringMVC入门](https://blog.lyu3.com/SpringMVC入门/)

本文所讲的部分内容跟SpringMVC的视图机制有关，SpringMVC的视图机制请参考楼主的另一篇博客：[SpringMVC视图机制详解](https://blog.lyu3.com/SpringMVC视图机制详解/)

---
<!-- more -->

## RedirectView介绍

RedirectView这个视图是跟重定向相关的，也是重定向问题的核心，我们来看看这个类的源码。

RedirectView属性：

![](http://images.cnitblog.com/i/411512/201406/141131199833771.png)

几个重要的构造方法：

![](http://images.cnitblog.com/i/411512/201406/141226085617324.png)

RedirectView渲染过程：

![](http://images.cnitblog.com/i/411512/201406/150016467026274.png)

重点看来下路径的构造：

![](http://images.cnitblog.com/i/411512/201406/150024082336531.png)

路径构造完毕之后使用reponse进行sendRedirect操作。

---

## 实例讲解

Controller代码：

```Java
@Controller
@RequestMapping(value = "/redirect")
public class TestRedirectController {
  
    @RequestMapping("/test1")
    public ModelAndView test1() {
        view.setViewName("redirect:index");
        return view;
    }

    @RequestMapping("/test2")
    public ModelAndView test2() {
        view.setViewName("redirect:login");
        return view;
    }
    
    @RequestMapping("/test3")
    public ModelAndView test3(ModelAndView view) {
        view.setViewName("redirect:/index");
        return view;
    }

    @RequestMapping("/test4")
    public ModelAndView test4(ModelAndView view) {
        view.setView(new RedirectView("/index", false));
        return view;
    }

    @RequestMapping("/test5")
    public ModelAndView test5(ModelAndView view) {
        view.setView(new RedirectView("index", false));
        return view;
    }

    @RequestMapping("/test6/{id}")
    public ModelAndView test6(ModelAndView view, @PathVariable("id") int id) {
        view.setViewName("redirect:/index{id}");
　　　　view.addObject("test", "test");
        return view;
    }
    
    @RequestMapping("/test7/{id}")
    public ModelAndView test7(ModelAndView view, @PathVariable("id") int id) {
        RedirectView redirectView = new RedirectView("/index{id}");
        redirectView.setExpandUriTemplateVariables(false);
        redirectView.setExposeModelAttributes(false);
        view.setView(redirectView);
        view.addObject("test", "test");
        return view;
    }
  
}
```

先看test1方法，返回值 "redirect:index" ， 那么会使用重定向。

SpringMVC找视图名"redirect:index"的时候，本文使用的ViewResolver是FreeMarkerViewResolver。

FreeMarkerViewResolver解析视图名的话，最调用父类之一的UrlBasedViewResolver中的createView方法。

![](http://images.cnitblog.com/i/411512/201406/141222324207258.png)

通过构造方法发现，这个RedirectView使用相对路径，兼容Http1.0，不暴露路径变量。

test1方法，进入的路径： /SpringMVCDemo/redirect/test1。  RedirectView使用相对路径，那么重定向的路径： /SpringMVCDemo/redirect/index

我们通过firebug看下路径：

![](http://images.cnitblog.com/i/411512/201406/150141294205639.png)

nice，验证了我们的想法。

test2方法同理：进入的路径： /SpringMVCDemo/redirect/test2。  重定向的路径： /SpringMVCDemo/redirect/login。

![](http://images.cnitblog.com/i/411512/201406/150141412957434.png)

test3方法：以 "/" 开头并且使用相对路径，那么会默认加上contextPath。 进入的路径： /SpringMVCDemo/redirect/test3。  重定向的路径： /SpringMVCDemo/index。

![](http://images.cnitblog.com/i/411512/201406/150141491706540.png)

test4方法：不使用默认路径，createTargetUrl方法中直接append  "/index"。进入的路径： /SpringMVCDemo/redirect/test4。  重定向的路径： /index。

![](http://images.cnitblog.com/i/411512/201406/150143029207737.png)

test5方法：不使用默认路径，createTargetUrl方法中直接append  "index"。进入的路径： /SpringMVCDemo/redirect/test5。  重定向的路径： /SpringMVCDemo/redirect/index。

其实这里有点意外，刚开始看的时候以为不使用绝对路径，以后路径会是/SpringMVCDemo/index或/index。 结果居然不是这样，感觉SpringMVC这个取名有点尴尬，有点蛋疼。 其实RedirectView中的createTargetUrl方法就明白了，源码是最好的文档 0 0.

![](http://images.cnitblog.com/i/411512/201406/150145299056276.png)

test6方法：使用默认路径，该方法还使用了路径变量。本文之前分析的时候说了RedirectView中构造重定向路径的时候会对路径变量进行替代，还会暴露model中的属性，且这2个暴露分别受属性expandUriTemplateVariables、exposeModelAttributes影响，这2个属性默认都是true。进入的路径： /SpringMVCDemo/redirect/test6/1。  重定向的路径： /SpringMVCDemo/index1?test=test。

![](http://images.cnitblog.com/i/411512/201406/150156234521805.png)

test7方法：跟test6方法一样，只不过我们不暴露model属性，不替代路径变量了。进入的路径： /SpringMVCDemo/redirect/test7/1。  重定向的路径： /SpringMVCDemo/index{id}。

![](http://images.cnitblog.com/i/411512/201406/150200273276543.png)

---

## 总结

简单了分析了RedirectView视图，并分析了该视图的渲染源码，并分析了重定向中的路径问题，参数问题，路径变量问题等常用的问题。

源码真是最好的文档，了解了视图机制之后，再回过头来看看RedirectView视图，So easy。

最后感觉RedirectView的相对路径属性怪怪的，不使用相对路径，"/" 开头的直接就是服务器根路径， 不带 "/" 开头的，又是相对路径， 有点蛋疼。

希望本文能够帮助读者了解SpringMVC的重定向相关问题。
