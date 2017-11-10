title: Servlet容器Tomcat中web.xml中url-pattern的配置详解[附带源码分析]
date: 2017-08-01
comments: true
tag:
 - Servlet
 - Tomcat
 - Java
categories: Java

----------
## 前言
今天研究了一下tomcat上web.xml配置文件中url-pattern的问题。

这个问题其实毕业前就困扰着我，当时忙于找工作。 找到工作之后一直忙，也就没时间顾虑这个问题了。 说到底还是自己懒了，没花时间来研究。

今天看了tomcat的部分源码 了解了这个url-pattern的机制。  下面让我一一道来。

tomcat的大致结构就不说了， 毕竟自己也不是特别熟悉。 有兴趣的同学请自行查看相关资料。 等有时间了我会来补充这部分的知识的。 

想要了解url-pattern的大致配置必须了解org.apache.tomcat.util.http.mapper.Mapper这个类

这个类的源码注释：Mapper, which implements the servlet API mapping rules (which are derived from the HTTP rules).  意思也就是说  “Mapper是一个衍生自HTTP规则并实现了servlet API映射规则的类”。

## 现象
首先先看我们定义的几个Servlet：
```xml
<servlet>
    <servlet-name>ExactServlet</servlet-name>
    <servlet-class>org.format.urlpattern.ExactServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ExactServlet</servlet-name>
    <url-pattern>/exact.do</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>ExactServlet2</servlet-name>
    <servlet-class>org.format.urlpattern.ExactServlet2</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ExactServlet2</servlet-name>
    <url-pattern>/exact2.do</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>TestAllServlet</servlet-name>
    <servlet-class>org.format.urlpattern.TestAllServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>TestAllServlet</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>TestServlet</servlet-name>
    <servlet-class>org.format.urlpattern.TestServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>TestServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
有4个Servlet。 分别是2个精确地址的Servlet：ExactServlet和ExactServlet2。 1个urlPattern为 “/*” 的TestAllServlet，1个urlPattern为 "/" 的TestServlet。

我们先来看现象：

![](http://images.cnitblog.com/i/411512/201405/051956554633273.jpg)
![](http://images.cnitblog.com/i/411512/201405/051957029954105.jpg)
![](http://images.cnitblog.com/i/411512/201405/051957096354968.jpg)
![](http://images.cnitblog.com/i/411512/201405/051957158237058.jpg)

test.do这个地址并不存在，因为没有相应的精确的urlPattern。  所以tomcat选择urlPattern为 "/*" 的Servlet进行处理。

index.jsp(这个文件tomcat是存在的)， 也被urlPattern为 "/*" 的Servlet进行处理。

 

我们发现，精确地址的urlPattern的优先级高于/*，　"/" 规则的Servlet没被处理。

为什么呢？　开始分析源码。

## 源码分析
本次源码使用的tomcat版本是7.0.52.

tomcat在启动的时候会扫描web.xml文件。 WebXml这个类是扫描web.xml文件的，然后得到servlet的映射数据servletMappings。

![](http://images.cnitblog.com/i/411512/201405/052010562455068.jpg)

然后会调用Context(实现类为StandardContext)的addServletMapping方法。 这个方法会调用本文开头提到的Mapper的addWrapper方法，这个方法在源码Mapper的360行。

![](http://images.cnitblog.com/i/411512/201405/052046370571633.jpg)

这里，我们可以看到路径分成4类。

1.  以 /* 结尾的。 path.endsWith("/*")

2.  以 *. 开头的。 path.startsWith("*.")

3.  是否是 /。      path.equals("/")

4.  以上3种之外的。 

各种对应的处理完成之后，会存入context的各种wrapper中。这里的context是ContextVersion，这是一个定义在Mapper内部的静态类。

![](http://images.cnitblog.com/i/411512/201405/052041278545902.jpg)

它有4种wrapper。 defaultWrapper，exactWrapper， wildcardWrappers，extensionWrappers。  

这里的Wrapper概念：

　　Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。

回过头来看mapper的addWrapper方法：

1. 我们看到  /* 对应的Servlet会被丢到wildcardWrappers中

2. *. 会被丢到extensionWrappers中

3. / 会被丢到defaultWrapper中

4. 其他的映射都被丢到exactWrappers中

最终debug看到的这些wrapper也验证了我们的结论。

![](http://images.cnitblog.com/i/411512/201405/052106586988983.jpg)

这里多了2个扩展wrapper，tomcat默认给我们加入的，分别处理.jsp和.jspx。

好了。 在这之前都是tomcat启动的时候做的一些工作。

下面开始看用户请求的时候tomcat是如何工作的：

 

用户请求过来的时候会调用mapper的internalMapWrapper方法， Mapper源码830行。

```java
// Rule 1 -- Exact Match
        Wrapper[] exactWrappers = contextVersion.exactWrappers;
        internalMapExactWrapper(exactWrappers, path, mappingData);

        // Rule 2 -- Prefix Match
        boolean checkJspWelcomeFiles = false;
        Wrapper[] wildcardWrappers = contextVersion.wildcardWrappers;
        if (mappingData.wrapper == null) {
            internalMapWildcardWrapper(wildcardWrappers, contextVersion.nesting,
                                       path, mappingData);
            .....
        }

        ....// Rule 3 -- Extension Match
        Wrapper[] extensionWrappers = contextVersion.extensionWrappers;
        if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
            internalMapExtensionWrapper(extensionWrappers, path, mappingData,
                    true);
        }

        // Rule 4 -- Welcome resources processing for servlets
        if (mappingData.wrapper == null) {
            boolean checkWelcomeFiles = checkJspWelcomeFiles;
            if (!checkWelcomeFiles) {
                char[] buf = path.getBuffer();
                checkWelcomeFiles = (buf[pathEnd - 1] == '/');
            }
            if (checkWelcomeFiles) {
                for (int i = 0; (i < contextVersion.welcomeResources.length)
                         && (mappingData.wrapper == null); i++) {
                    ...// Rule 4a -- Welcome resources processing for exact macth
                    internalMapExactWrapper(exactWrappers, path, mappingData);

                    // Rule 4b -- Welcome resources processing for prefix match
                    if (mappingData.wrapper == null) {
                        internalMapWildcardWrapper
                            (wildcardWrappers, contextVersion.nesting,
                             path, mappingData);
                    }

                    // Rule 4c -- Welcome resources processing
                    //            for physical folder
                    if (mappingData.wrapper == null
                        && contextVersion.resources != null) {
                        Object file = null;
                        String pathStr = path.toString();
                        try {
                            file = contextVersion.resources.lookup(pathStr);
                        } catch(NamingException nex) {
                            // Swallow not found, since this is normal
                        }
                        if (file != null && !(file instanceof DirContext) ) {
                            internalMapExtensionWrapper(extensionWrappers, path,
                                                        mappingData, true);
                            if (mappingData.wrapper == null
                                && contextVersion.defaultWrapper != null) {
                                mappingData.wrapper =
                                    contextVersion.defaultWrapper.object;
                                mappingData.requestPath.setChars
                                    (path.getBuffer(), path.getStart(),
                                     path.getLength());
                                mappingData.wrapperPath.setChars
                                    (path.getBuffer(), path.getStart(),
                                     path.getLength());
                                mappingData.requestPath.setString(pathStr);
                                mappingData.wrapperPath.setString(pathStr);
                            }
                        }
                    }
                }

                path.setOffset(servletPath);
                path.setEnd(pathEnd);
            }

        }

        /* welcome file processing - take 2
         * Now that we have looked for welcome files with a physical
         * backing, now look for an extension mapping listed
         * but may not have a physical backing to it. This is for
         * the case of index.jsf, index.do, etc.
         * A watered down version of rule 4
         */
        if (mappingData.wrapper == null) {
            boolean checkWelcomeFiles = checkJspWelcomeFiles;
            if (!checkWelcomeFiles) {
                char[] buf = path.getBuffer();
                checkWelcomeFiles = (buf[pathEnd - 1] == '/');
            }
            if (checkWelcomeFiles) {
                for (int i = 0; (i < contextVersion.welcomeResources.length)
                         && (mappingData.wrapper == null); i++) {
                    path.setOffset(pathOffset);
                    path.setEnd(pathEnd);
                    path.append(contextVersion.welcomeResources[i], 0,
                                contextVersion.welcomeResources[i].length());
                    path.setOffset(servletPath);
                    internalMapExtensionWrapper(extensionWrappers, path,
                                                mappingData, false);
                }

                path.setOffset(servletPath);
                path.setEnd(pathEnd);
            }
        }


        // Rule 7 -- Default servlet
        if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
            if (contextVersion.defaultWrapper != null) {
                mappingData.wrapper = contextVersion.defaultWrapper.object;
                mappingData.requestPath.setChars
                    (path.getBuffer(), path.getStart(), path.getLength());
                mappingData.wrapperPath.setChars
                    (path.getBuffer(), path.getStart(), path.getLength());
            }
            ...
        }
```

这段代码作者已经为我们写好了注释.

Rule1，Rule2，Rule3....

看代码我们大致得出了：

用户请求这里进行url匹配的时候是有优先级的。 我们从上到下以优先级的高低进行说明：

规则1：精确匹配，使用contextVersion的exactWrappers

规则2：前缀匹配，使用contextVersion的wildcardWrappers

规则3：扩展名匹配，使用contextVersion的extensionWrappers

规则4：使用资源文件来处理servlet，使用contextVersion的welcomeResources属性，这个属性是个字符串数组

规则7：使用默认的servlet，使用contextVersion的defaultWrapper

 

最终匹配到的wrapper（其实也就是servlet）会被丢到MappingData中进行后续处理。

 

下面验证我们的结论：

我们在配置文件中去掉 /* 的TestAllServlet这个Servlet。 然后访问index.jsp。 这个时候规则1精确匹配没有找到，规则2前缀匹配由于去掉了TestAllServlet，因此为null，规则3扩展名匹配(tomcat自动为我们加入的处理.jsp和.jspx路径的)匹配成功。最后会输出index.jsp的内容。

![](http://images.cnitblog.com/i/411512/201405/052109008853746.jpg)

我们再来验证http://localhost:7777/UrlPattern_Tomcat/地址。（TestAllServlet依旧不存在）

![](http://images.cnitblog.com/i/411512/201405/052242129798562.png)

规则1，2前面已经说过，规则3是.jsp和.jspx。 规则4使用welcomeResources，这是个字符串数组，通过debug可以看到

![](http://images.cnitblog.com/i/411512/201405/052229135102597.png)

会默认取这3个值。最终会通过规则4.c匹配成功，这部分大家可以自己查看源码分析。

最后我们再来验证一个例子：

将TestAllServlet的urlpattern改为/test/*。

| 地址 | 匹配规则情况 | Servlet类 |
|---|
|http://localhost:7777/UrlPattern_Tomcat/exact.do|规则1，精确匹配没有找到|ExactServlet|
|http://localhost:7777/UrlPattern_Tomcat/exact2.do|规则1，精确匹配没有找到|ExactServlet2|
|http://localhost:7777/UrlPattern_Tomcat/test/index.jsp|规则2，前缀匹配找到|TestAllServlet|
|http://localhost:7777/UrlPattern_Tomcat/index.jsp （规则2已经匹配到，这里没有进行匹配）|规则3，扩展名匹配没有找到|TestServlet|

![](http://images.cnitblog.com/i/411512/201405/052248308544372.png)

验证成功。

## 实战例子
SpringMVC相信大家基本都用过了。 还不清楚的同学可以看看它的入门blog：
[https://blog.lyu3.com/SpringMVC入门/](https://blog.lyu3.com/SpringMVC入门/)

SpringMVC是使用DispatcherServlet做为主分发器的。  这个Servlet对应的url-pattern一般都会用“/”，当然用"/*"也是可以的，只是可能会有些别扭。

 

如果使用/*，本文已经分析过这个url-pattern除了精确地址，其他地址都由这个Servlet执行。

比如这个http://localhost:8888/SpringMVCDemo/index.jsp那么就会进入SpringMVC的DispatcherServlet中进行处理，最终没有没有匹配到 /index.jsp 这个 RequestMapping， 解决方法呢  就是配置一个：
![](http://images.cnitblog.com/i/411512/201405/071533542765836.jpg)

最终没有跳到/webapp下的index.jsp页面，而是进入了SpringMVC配置的相应文件("/*"的优先级比.jsp高):

![](http://images.cnitblog.com/i/411512/201405/071535025426858.jpg)

## 总结
之前这个url-pattern的问题自己也上网搜过相关的结论，网上的基本都是结论，自己看过一次之后过段时间就忘记了。说到底还是不知道工作原理，只知道结论。而且刚好这方面的源码分析类型博客目前还未有人写过，于是这次自己也是决定看看源码一探究竟。

总结： 想要了解某一机制的工作原理，最好的方法就是查看源码。然而查看源码就需要了解大量的知识点，这需要花一定的时间。但是当你看明白了那些源码之后，工作原理也就相当熟悉了， 就不需要去背别人写好的一些结论了。 建议大家多看看源码。
