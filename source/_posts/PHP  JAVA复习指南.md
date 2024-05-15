---
title: PHP / JAVA复习指南
date: 2022-11-03 23:51:24
excerpt: PHP / JAVA复习指南
categories: 学习
---

# PHP / JAVA复习指南

## PHP

### 1.php 命令执行函数以及函数应用场景

- `system()` 将字符串作为OS命令执行，自带输出功能。
- `passthru()`将字符串作为OS命令执行，不需要输出执行结果，且输出全部的内容。
- `exec()`将字符串作为OS命令执行，需要输出执行结果，且它只会输出最后一行的内容。
- `shell_exec` 将字符串作为OS命令执行，需要输出执行结果，且输出全部的内容
- `popen()/proc_open()`该函数也可以将字符串当作OS命令来执行，但是该函数返回的是文件指针而非命令执行结果。该函数有两个参数。需要用**<u>fread</u>**来读
- `反引号 ` [``]反引号里面的代码也会被当作OS命令来执行

### 2.php伪协议以及各个伪协议的应用场景以及应用的条件是什么？需要php.ini中进行怎样的配置？

php封装协议又称为伪协议，主要分为以下几种：

- `file://`  file:// 用于访问本地文件系统它在双off的情况下也可以正常使用；allow_url_fopen ：off/on allow_url_include：off/on

- `php://filter` 读取源代码并进行base64编码输出，不然会直接当做php代码执行就看不到源代码内容了在双off的情况下也可以正常使用；

- `php://input` 可以访问请求的原始数据的只读流。即可以直接读取到POST上没有经过解析的原始数据allow_url_fopen ：off/on

  allow_url_include：on

- `zip:// compress.bzip:// compress.zlib://`均属于压缩流，可以访问压缩文件中的子文件，更重要的是不需要指定后缀名。双off也能用

- `data://` 数据流封装器开始有效，主要用于数据流的读取，如果传入的数据是PHP代码就会执行代码。必须要双on才可以

- `phar:/`/ 反序列化有时会用到 与zip协议不同的是zip协议为绝对路径，而phar协议为相对路径

### 3.代码执行函数

- `preg_replace()` 在php 5.6及之前的版本，当第一个参数在 /e 的修饰下，并且 magic_quotes_gpc=Off 时，函数的第二个参数会被当做php代码执行

  ```
  <?php
  	preg_replace('/<php>(.*?)' . $_GET['reg'], '\\1' ,'<php>phpinfo()<php>');
  	preg_replace('/anything/e', $_GET['test'], 'anything_test');
  	preg_replace('/<php>(.*?)<\/php>/e', '\\1', $_GET['test']);
  ```

- `array_map()` 第一个参数是回调函数，第二参数是函数的参数

  ```
  <?php
      $func=$_GET['func'];
      $cmd=$_POST['cmd'];
      $array[0]=$cmd;
      $new_array=array_map($func,$array);
      echo $new_array;
  ?>
  //    命令执行http://localhost/123.php?func=system   cmd=whoami
   //   菜刀连接http://localhost/123.php?func=assert   密码：cmd
  ```

  

- `assert()` 判断一个表达式是否成立，能够把字符组成的字符串，当作代码执行。

  > assert函数是直接将传入的参数当成PHP代码直接，不需要以分号结尾（特别注意），有时加上分号不会显示结

### 4.php 中常见的文件读取/写入函数

共有9个文件读取函数

```
file_get_contents   // *
fread
fgets
fgetss
file
parse_ini_file
readfile    // *
highlight_file  // *
show_source // *
```

使用方法

```php
<?php

// file_get_contents
print(sprintf("%'-10s%-'-30s", '-', 'file_get_contents').PHP_EOL);
echo file_get_contents('flag.txt');
echo PHP_EOL;

// fopen fread
print(sprintf("%'-10s%-'-30s", '-', 'fopen fread').PHP_EOL);
$file = fopen("flag.txt","rb");
echo fread($file,1024);     // 参数为 resource 类型
fclose($file);
echo PHP_EOL;

// fopen fgets
print(sprintf("%'-10s%-'-30s", '-', 'fopen fgets').PHP_EOL);
$file = fopen("flag.txt","r");      
echo fgets($file, 4096);        // 过滤掉了 HTML 和 PHP 标签
fclose($file);
echo PHP_EOL;

// fopen fgetss
print(sprintf("%'-10s%-'-30s", '-', 'fopen fgetss').PHP_EOL);
$file = fopen("flag.txt","r");     
echo fgetss($file, 4096);        // 过滤掉了 HTML 和 PHP 标签
fclose($file);
echo PHP_EOL;

// readfile
print(sprintf("%'-10s%-'-30s", '-', 'readfile').PHP_EOL);
echo readfile("flag.txt");      // 看到不仅输出了所有内容，而且还输出了总共长度
echo PHP_EOL;

// file
print(sprintf("%'-10s%-'-30s", '-', 'file').PHP_EOL);
print_r(file('flag.txt'));      // 读取结果为数组，所以需要用 print_r 或 var_dump 
echo PHP_EOL;

// parse_ini_file
print(sprintf("%'-10s%-'-30s", '-', 'parse_ini_file').PHP_EOL);
echo parse_ini_file("flag.txt");        // 只能读取 ini 配置文件
echo PHP_EOL;

// show_source
print(sprintf("%'-10s%-'-30s", '-', 'show_source').PHP_EOL);
show_source('flag.txt');
echo PHP_EOL;

// highlight_file
print(sprintf("%'-10s%-'-30s", '-', 'highlight_file').PHP_EOL);
highlight_file('flag.txt');
echo PHP_EOL;


?>

```

在PHP中使用 fweite() 和file_put_contents()函数向文件中写入数据。

```php
<?php
$file = fopen("test.txt","w");
echo fwrite($file,"Hello World. Testing!");
fclose($file);
?>

<?php
echo file_put_contents("test.txt","Hello World. Testing!");
?>
```



### 5.php 文件包含方法以及区别

- `include`

  ```
  <?php include 'test.php';?>
  ```

- `include_once`  可以用于在脚本执行期间同一个文件有可能被包含超过一次的情况下，想确保它只被包含一次以避免函数重定义，变量重新赋值等问题

- `require` 与include方法基本一样

- `require_once`   require_once 语句时会先检查要包含的文件是不是已经在该程序中的其他地方被包含过，如果有，则不会再次重复包含该文件

区别

> 1、include如果碰到错误，会给出提示，并继续向下执行；而require会终止程序执行。2、require_once和include_once中如果包含的文件已经被包含过，就不会再次包含，但include和require会。



### 6.php反序列化漏洞涉及函数

- `serialize()`  序列化：把对象转化为二进制的字符串

- `unserialize()` 反序列化：把对象转化的二进制字符串再转化为对象

- 魔术方法 可能在某些情况下自动调用

  ```
  1.__construct()，类的构造函数
  
  2.__destruct()，类的析构函数
  
  3.__call()，在对象中调用一个不可访问方法时调用
  
  4.__callStatic()，用静态方式中调用一个不可访问方法时调用
  
  5.__get()，获得一个类的成员变量时调用
  
  6.__set()，设置一个类的成员变量时调用
  
  7.__isset()，当对不可访问属性调用isset()或empty()时调用
  
  8.__unset()，当对不可访问属性调用unset()时被调用。
  
  9.__sleep()，执行serialize()时，先会调用这个函数
  
  10.__wakeup()，执行unserialize()时，先会调用这个函数
  
  11.__toString()，类被当成字符串时的回应方法
  
  12.__invoke()，调用函数的方式调用一个对象时的回应方法
  
  13.__set_state()，调用var_export()导出类时，此静态方法会被调用。
  
  14.__clone()，当对象复制完成时调用
  
  15.__autoload()，尝试加载未定义的类
  
  16.__debugInfo()，打印所需调试信息
  ```

### 7.php 常见数组及引用

- `$_GET` 预定义的 $_GET 变量用于收集来自 method="get" 的表单中的值。

  从带有 GET 方法的表单发送的信息，对任何人都是可见的（会显示在浏览器的地址栏），并且对发送信息的量也有限制

- `$POST`预定义的 $_POST 变量用于收集来自 method="post" 的表单中的值。

  从带有 POST 方法的表单发送的信息，对任何人都是不可见的（不会显示在浏览器的地址栏），并且对发送信息的量也没有限制。

- `$_REQUEST` 既可以接收$_GET又可以接收$POST的值

- `$_FILES` 上传文件信息

- `$_COOKIE`用于获取 cookie 的值。

- `$SESSION`读取Session变量信息

## JAVA

### 1.Web服务器

常见的java EE web服务器

**tomcat   jetty  resin**

专业级java EE web 服务器

**JBOSS   GlassFish   Weblogic   Webshpere**



常见漏洞

https://zhuanlan.zhihu.com/p/399939177

### 2.tomcat目录结构与功能

- `bin`:目录里主要是有启动和关闭应用服务器的bat批处理命令；
- `conf`:这个目录里主要是支持配置Tomcat的文件。
- `lib`:这里面是支持Tomcat启动的jar包，当然也可以放你自己项目需要的jar包。
- `temp`:Tomcat运行过程中产生的临时文件。
- `webapps`:这里就是你的项目了，你的项目可以是以文件或者jar包的方式存放在这个目录里。
- `work`：work目录用来存放tomcat在运行时的编译后文件，例如JSP编译后的文件。 



### 3.tomcat中web.xml 与 tomcat_user.xml 的作用

`web.xml`
Tomcat可以让用户通过将缺省的web.xml放入conf目录中来定义所有关系环境的web.xml的缺省值.建立一个新的关系环境时,Tomcat使用缺省的web.xml文件作为基本设置和应用项目特定的web.xml(放在应用项目的WEB-INF/web.xml文件)来覆盖这些缺省值.是配置整个tomcat的jsp和servlet工作中的一些情况,比如我们配置list来不让我们输入一个目录的时候显示出那个目录下的jsp文件,而是显示404错误.还有在一些安全方面也可以做配置.

`tomcat-users.xml`
 配置tomcat的用户了信息.可以到tomcat的开始页http://localhost:8080中点tomcat manager就会提示你要用户名和密码了,这里的用户名和密码就可以在这个xml中配置的



### 4.什么是jsp

> JSP全称Java Server Pages，是一种动态网页开发技术。它使用JSP标签在HTML网页中插入Java代码。标签通常以<%开头以%>结束。
>
> JSP是一种Java servlet，主要用于实现Java web应用程序的用户界面部分。网页开发者们通过结合HTML代码、XHTML代码、XML元素以及嵌入JSP操作和命令来编写JSP。
>
> JSP通过网页表单获取用户输入数据、访问数据库及其他数据源，然后动态地创建网页。
>
> JSP标签有多种功能，比如访问数据库、记录用户选择信息、访问JavaBeans组件等，还可以在不同的网页中传递控制信息和共享信息。



### 5.java 代码审计xss

```
前端反射型xss   (name参数可控) 
<%
String name = request.getParameter("name");
out.println(name)
%>
后端反射型 xss  (msg参数可控) 
public void Message(HttpServletRequest req, HttpServletResponse resp) {
	String message = req.getParameter("msg");
try{
	resp.getWriter().print(message);
} catch (IOException e) {
	e. printStackTrace();
}
DOM xss只发生在客户端处理数据阶段，未经过滤被传入存在缺陷的JavaScript代码处理。
<script>
    var pos = document.URL.indexOf("#")+1;
    var name = document.URL.substring(pos, document.URL.length);
    document.write(name);
eval("var a = " + name);
</script>
储存型xss 会通过JDBC与数据库交互
```



### 6.Java 常用的文件操作方法

```
Files.readAllBytes、Files.readAllLines
FileInputStream
FileOutputStream
File
FileUtils
IOUtils
BufferedReader
ServletFileUpload
MultipartFile
CommonsMultipartFile
PrintWriter
ZipInputStream
ZipEntry.getSize
```



### 7.文件上传关键函数

```
DiskFileItemFactory@MultipartConfigMultipartFileFile
Upload
InputStream
OutputStream
write
fileName
filePath
```



### 8.servlet文件上传函数

> servlet3之前
> 使用commons-fileupload、commons-io这两个jar包来处理文件上传
>
> servlet3之后
> 使用request.getParts()获取上传文件



### 9.java任意文件上传

```
private boolean uploadWithAnnotation(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
    Part part = request.getPart("fileName");
    if(part == null) {
      return false;
    }
    String filename = UPLOAD_PATH + File.separator + part.getSubmittedFileName();
    part.write(filename);
    part.delete();
    return true;
}
```



### 10.任意文件删除

```
String filename = request.getParameter("filename");
File file = new File(filename);
if(file != null && file.exists() && file.delete()) {
  	response.getWriter().println("delete success");
} else {
  	response.getWriter().println("delete failed");
}
```