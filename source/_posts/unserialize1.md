---
title: 初识反序列化
date: 2022/9/15 12:00:00
cover: https://i.postimg.cc/NMWzHWLm/us1.jpg
categories:
- CTF
tags:
- web
- 反序列化
---

# 反序列化

## 序列化和反序列化的概念

在各类语言中，**将对象的状态信息转换为可储存或可传输的过程**就是序列化，而其逆过程就是反序列化

> bool : b:value => b:0
>
> int : i:value => i:1
>
> str : s:length:"value" => s:4:"aaaa"
>
> array : a:\<length>:{key,value pairs}; => a:1:{i:1,s:1:"a"}
>
> object : O:<class_name_legth>:
>
> NULL型 : N

而最终的序列化形式为 `<class_name>:<number_of_properties>:{<properties>};`

例如：

```
O:3:"Ctf":3{s:4:"flag";s:13:"flag{abedyui}";s:4:"name";s:7:"Sch0lar";s:3:"age";s:2:"18";}

O代表对象 因为我们序列化的是一个对象 序列化数组则用A来表示
3 代表类名字占三个字符 
ctf 类名
3 代表三个属性
s代表字符串
4代表属性名长度
flag属性名
s:13:"flag{abedyui}" 字符串 属性值长度 属性值
```

利用思路：PHP中存在魔法方法，可以自身调用

**常见的魔法函数：**

```php
__construct()   创建对象时调用
__destruct()    销毁对象时调用
__toString() 	当一个对象被当作一个字符串使用
__sleep() 		在对象在被序列化之前运行
__wakeup() 		将在序列化之后立即被调用
__call() 		反序列化恢复对象前调用
__get()  		从不可访问的属性读取数据
```

## 绕过__wakeup()

原理：**当序列化字符串表示对象属性个数的值大于真实个数的属性时就会跳过__wakeup的执行。**

```
//将上面的对象属性个数值改成逼真实个数打
O:4:"Name":3:{s:14:"%00Name%00username";s:5:"admin";s:14:"%00Name%00password";i:100;}
```

例：

```php
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}
?code=
```

1. **\_wakeup()**是一个魔法方法，_wakeup()会**直接退出**，并输出" bad requests "

2. 末尾有"?code= " ，因此要构造url来绕过_wakeup()这个函数。

最后将其序列化：

```php
<?php
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}}
//?code=
$a = new xctf();

//序列化数组
$s = serialize($a);
echo $s;
//输出结果：a:3:<?php
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}}
//?code=
$a = new xctf();

//序列化数组
$s = serialize($a);
echo $s;
/*输出结果：
	O:4:"xctf":1:{s:4:"flag";}
	此时构造payload：O:4:"xctf":1:{s:4:"flag";s:3:"111";}
*/
?>
```

## 反序列化的绕过

- 绕过 `/[oc]:\d+:/i` 过滤，`/[oc]:\d+:/i` 会过滤 `O：d` 因此我们需要在之间加一个 `+` 以退出匹配来,例如 `O:+d`