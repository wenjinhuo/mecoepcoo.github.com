---
title: PHP表单验证
date: 2016-05-13 15:56:22
tags: 
      - 后台
      - php
      - 技术
toc: true
---

# PHP表单验证

本文介绍使用PHP连接数据库进行表单验证的方法。

[TOC]

## 1. 准备工作

### 1.1 数据表

在名称为demo的数据库中建立表名为demo_admin的数据表如下：

| 字段名      | 数据类型    | 长度   | 小数点  | 是否允许空 |      |
| -------- | ------- | ---- | ---- | ----- | ---- |
| id       | int     | 15   | 0    | 否     | 主键   |
| username | varchar | 30   | 0    | 否     |      |
| password | char    | 30   | 0    | 否     |      |
| email    | varchar | 60   | 0    | 否     |      |

建表的方法不在此文赘述。

在表中添加一条记录：

id=>1

username=>admin

password=>827ccb0eea8a706c4c34a16891f84e7b      //12345的md5加密形式

email=>123@qq.com

<!--more-->

### 1.2 前台表单

在前台 **login.php** 建立如下表单：

```HTML
<form action="doLogin.php" method="post">
  <ul class="login">
    <li class="">邮箱/用户名/手机号</li>
    <li class="">
      <input type="text" class="" name="username">
    </li>
    <li class="">密码</li>
    <li class="">
      <input type="text" class="" name="password">
    </li>
    <li class="l_tit">验证码</li>
      <img src="getVerify.php" alt="verify"/>
    <li class="">
      <input type="text" class="" name="verify">
    </li>
    <li>
      <input type="submit" value="提交" class="">
    </li>
  </ul>
</form>
```

getVerify.php为验证码生成程序，相关内容见 《PHP通过GD库制作简单验证码》 

### 1.3 文件目录

项目目录结构如下：

www/admin/**login.php**        //登录前台页面

www/admin/**doLogin.php**        //登录操作程序

www/configs/**configs.php**        //数据库配置文件

www/core/**admin.inc.php**        //定义验证方法

www/lib/**mysql.func.php**        //数据库连接及sql语句封装文件

www/**include.php**        //引用文件

## 2. 项目实现

### 2.1 configs.php

编写数据库配置文件：

```php
[PHP]
//数据库位置为localhost
define("DB_HOST","localhost");
//数据库登录用户名为root
define("DB_USER","root");
//数据库密码为root
define("DB_PWD","root");
//数据库名为demo
define("DB_DBNAME","demo");
//数据库编码格式为utf8
define("DB_CHARSET","utf8");
```

### 2.2 mysql.func.php

连接数据库，并封装一些常用的sql方法：

```php
[PHP]
<?php
require_once '../include.php';
//连接数据库
function connect(){
  $link=mysql_connect(DB_HOST,DB_USER,DB_PWD) or die ("数据库连接失败Error：".mysql_errno().":".mysql_error());
  mysql_set_charset(DB_CHARSET);
  mysql_select_db(DB_DBNAME) or die("连接失败");
  return $link;
}
//封装一系列方法
//增加（插入）
function insert($table, $array){
  //数组的键名变成插入语句中的键名
  $keys=join(",",array_keys($array));
  //数组中的值变成插入记录的值
  $vals="'".join("','",array_values($array))."'";
  //sql语句
  $sql="insert {$table}{$keys} values({$vals})";
  mysql_query($sql);
  return mysql_insert_id();
}
//删除
function delete($table,$where=null){
  $where=$where==null?null:"where".$where;
  $sql="delete from {$table}{$where}";
  mysql_query($sql);
  return mysql_affected_rows();
}
//更新
//sql句型：update demo_admin set username='admin' where id = 1
function update($table,$array,$where=null){
  foreach($array as $key=>$val){
    if($str==null){
      $sep="";
    }else{
      $sep=",";
    }
    $str.=$sep.$key."='".$val."'";
  }
  $sql="update {$table} set {$str}".($where==null?null:"where".$where);
  mysql_query($sql);
  return mysql_affected_rows();
}
//查询指定的一条记录
function fetchOne($sql,$result_type=MYSQL_ASSOC){
  $result=mysql_query($sql);
  $row=@mysql_fetch_array($result,$result_type);
  return $row;
}
//查询结果集中的所有记录
function fetchAll($sql,$result_type=MYSQL_ASSOC){
  $result=mysql_query($sql);
  while(@$row=mysql_fetch_array($result,$result_type)){
    $rows[]=$row;
  }
  return $rows;
}
//得到结果集中的记录条数
function getResultNum($sql){
  $result=mysql_query($sql);
  return mysql_numrows($result);
}
```

### 2.5 admin.inc.php

代码如下：

```php
[PHP]
//查找数据库
function checkAdmin($sql){
  return fetchOne($sql);
}
```



### 2.4 doLogin.php

编写验证文件：

```php
[PHP]
require_once '../include.php';
session_start();
//接收传来的用户名
$username=$_POST['username'];
//接收传来的密码，并用md5加密
$password=md5($_POST['password']);
//接收传来的验证码
$verify=$_POST['verify'];
//接收session中的验证码
$verify1=$_SESSION['verify'];
//优先判断验证码的正确性，并做出相应操作
if($verify==$verify1){
  $sql="select * from demo_admin where username='{$username}' and password='{$password}'";
  $row=checkAdmin($sql);
  if($row){
    $_SESSION['adminName']=$row['username'];
    //显示登录成功，并跳转到index.html
    //header("location:index.php");
    alertMes("登录成功","index.html");
  }else{
    //此处为自行封装的方法，这里不再给出代码
    alertMes("登录失败，重新登录","login.php");
  }
}else{
  echo "<script>alert('验证码错误');</script>";
  echo "<script>window.location='login.php';</script>>";
}
```

封装的alertMes()方法：

```php
[PHP]
function alertMes($mes,$url){
  echo "<script>alert('{$mes}');</script>";
  echo "<script>window.location='{$url}';</script>>";
}
```



### 2.5 include.php  

编写include.php用以快速引入所有需要的文件：

```php
[PHP]
<?php
//指定网页编码为utf8
header("content-type:text/html;charset=utf-8");
//定义魔术变量，定义“__FILE__”为站点根目录
define("ROOT",dirname(__FILE__));
//设置include_path运行时的配置选项
set_include_path(".".PATH_SEPARATOR.ROOT."/lib".PATH_SEPARATOR.ROOT."/core".PATH_SEPARATOR.ROOT."/configs".PATH_SEPARATOR.get_include_path());
//引入文件
require_once 'configs.php';
require_once 'mysql.func.php';
require_once 'string.func.php';
require_once 'image.func.php';
require_once 'admin.inc.php';
//调用连接数据库方法
connect();
```

至此，PHP登录的表单验证demo完成，登录名为admin，密码为12345



> May 13， 2016 By 天真