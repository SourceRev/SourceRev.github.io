---
title: mysql基础
date: 2022/9/14 16:47:47
cover: https://i.postimg.cc/d0yTZRC6/mysql.jpg
categories:
- 开发
tags:
- MySQL
---

# 基本语法

## 外部语法

### 远程连接数据库

```mysql
//本机可填localhost
>mysql -uroot -p123456 -h 服务端地址
```

### 默认以utf8来创建数据库

```linux
>vim /etc/my.cnf
//在[mysqld]随即一行添加如下:
character_set_server=utf8
```

### 创建特殊的数据库名

1. 含有非法字符、保留关键字的处理方法

```mysql
#利用反引号将名字括起来
create database `test-abc`
#但实际是将字符转化为特殊符号进行储存
`test-abc` = test@cabc
```

2. 数据库名称中不能含有 `\` `/` `:` `?` `*` `<` `>` `|` 等字符
3. 数据库不能重名

## 内部语法

### 测试网站是否支持php

```
vim index.php//探针文件
<?php
	phpinfo();
?>
```

### 更改文件权限

```linux
# chown apache.apache 目标文件名/ -R
```

### 免登录执行指令

```mysql
mysql -uroot -p123456 -e "执行命令"
```

### 增加

```mysql
create database 数据库名	#创建一个新的数据库
show create database 数据库名	#查看创建的过程
create database 数据库名 CHARACTER SET utf8	#以utf-8的格式创建数据库
create database if not exists 数据库名; #如果不存在则创建该数据库

#字段：
insert into (数据库名.)表名 values (每一个字段所要赋的值);#数据库表中插入字段
insert into (数据库名.)表名 values (每一个字段所要赋的值),(同前);#数据库表中插入多个字段
```

### 查询

**select**:

```mysql
select database();#查询当前正在使用的数据库
select now();	  #输出系统当前时间
select user();	  #输出当前使用数据库的使用者
select * from (数据库名.)表名;#查询表中所有字段
select * from (数据库名.)表名\G;#查询表中所有字段(按行输出)
select * from 表名 where 字段 is null;#当字段赋值为空时进行的查询
select distinct 字段 from 表名;#去重查询
```

**desc**:

```mariadb
desc 表名; #查看表的结构
```

### 删除

**drop**:

```mysql
drop database 数据库名;  #删除数据库
drop database `数据库名`;#删除带特殊符号的数据库
drop database if exists 数据库名; #如果数据库存在，则删除它
```

**delete**:

```mariadb
delete from 表名 [where 筛选条件(例如：age=21 and id=3)];
delete * from 表名 where 字段 is null;#当字段赋值为空时进行的删除
```

### 修改

**alter**:

```mysql
alter table 表名 rename 字段; #修改数据库的表名
alter table 表名 modify 需要修改的字段 int(数值); #修改表中字段的长度（表的结构）
alter table 表名 change 旧字段 新字段 类型(例如：varchar(30));#修改表中的字段名

alter table 表名 add 新字段 enum(); #从表末新增表名，同时给表定义一个枚举的类型
alter table 表名 add 新字段 int(20) first; #从表头新增表名，同时给表定义整形
alter table 表名 add 新字段 int(20) after 字段; #从指定的位置后新增表名，同时给表定义整形
alter table 表名 drop 字段名;#删除字段，内容也被清除
```

### change和modify的区别

**CHANGE** 对列进行重命名和更改列的类型，需给定旧的列名称和新的列名称、当前的类型。

**MODIFY** 可以改变列的类型，此时不需要重命名（不需给定新的列名称）。

**update**:

```mariadb
update 表名 set 字段='123' where id=5;
```



# 基本信息

在mysql->user的表中存放root用户的许可信息，可以限制远程连接主机的权限。

## sql基本函数

**group_concat() 函数**：用于将SQL语句的结果拼接在一起，如果我们的查询结果多于一个就需要将这些结果拼接出来

## sql的基本认识

**information_schema 库**：这个库是在MySql 5.0之后的一个库，用来存放整个数据库的信息，里面可以查询到 所有的库名，表名，列名
