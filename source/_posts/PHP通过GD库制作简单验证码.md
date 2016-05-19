---
title: PHP通过GD库制作简单验证码
date: 2016-05-11 15:56:22
tags: 
      - 后台
      - php
      - 技术
toc: true
---

# PHP通过GD库制作简单验证码

**该项目的测试开发环境为win10 + Apache2.4.18 + PHP5.5.30**

[TOC]

## 1. 文件目录

站点根目录为site2，文件结构如下：

site2

- site2/admin/**login.php**
- site2/admin/**getVerify.php**


- site2/lib/**image.func.php**


- site2/lib/**string.func.php**

- site2/fonts/**STXINGKA.TTF**

  <!--more-->

## 2. 生成字符 

在 **string.func.php** 中定义生成字符的方法：

```php
[PHP]
function buildRandomString($type=1,$length=4){  //传入type，length两个参数
  //设置3种生成字符的方法
  if ($type == 1) {
    $chars = join ( "", range ( 0, 9 ) );
  } elseif ($type == 2) {
    $chars = join ( "", array_merge ( range ( "a", "z" ), range ( "A", "Z" ) ) );
  } elseif ($type == 3) {
    $chars = join ( "", array_merge ( range ( "a", "z" ), range ( "A", "Z" ), range ( 0, 9 ) ) );
  }
  //验证码长度不能大于字符串长度
  if ($length > strlen ( $chars )) {
    exit ( "字符串长度不够" );
  }
  //随机打乱所有字符
  $chars = str_shuffle ( $chars );
  //从字符串中的第一个字符开始返回，长度为length
  return substr ( $chars, 0, $length );
}
```

该方法生成了一个长度为4的字符串。

## 3. 生成验证码图像

### 3.1 生成图像

在 **image.func.php** 中生成图像验证码：

```php
[PHP]
//引入生成字符串文件
require_once 'string.func.php';
//启用session
session_start();
//通过GD库做验证码
//创建画布
$width = 80;
$height = 28;
//建立宽为width，高为height的画布
$image = imagecreatetruecolor ( $width, $height );
//设定一些颜色
$white = imagecolorallocate ( $image, 255, 255, 255 );
$black = imagecolorallocate ( $image, 0, 0, 0 );
//用填充矩形填充画布
imagefilledrectangle ( $image, 1, 1, $width - 2, $height - 2, $white );
//调用创建字符串方法并传递$type,$length参数
$type=1;
$length=4;
$chars = buildRandomString ( $type, $length );
//把字符串存入SESSION
$sess_name="verify";
$_SESSION [$sess_name] = $chars;
//设置一些字体
$fontfiles = array ("STXINGKA.TTF" );
//产生长度为length的字符图像
for($i = 0; $i < $length; $i ++) {
  $size = mt_rand ( 14, 18 );
  //角度-15到15
  $angle = mt_rand ( - 15, 15 );
  //字符显示的位置
  $x = 5 + $i * $size;
  $y = mt_rand ( 20, 26 );
  //从设置的字体文件中随机取一个字体
  $fontfile = "../fonts/" . $fontfiles [mt_rand ( 0, count ( $fontfiles ) - 1 )];
  //随机取一个颜色
  $color = imagecolorallocate ( $image, mt_rand ( 50, 90 ), mt_rand ( 80, 200 ), mt_rand ( 90, 180 ) );
  //截取，从i开始取，取1位
  $text = substr ( $chars, $i, 1 );
  //将TTF文字写到图中
  imagettftext ( $image, $size, $angle, $x, $y, $color, $fontfile, $text );
}
//绘制干扰点元素
$pixel=30;
if($pixel){
  for($i=0;$i<$pixel;$i++){
    imagesetpixel($image, mt_rand(0, $width-1),mt_rand(0, $height-1),$black);
  }
}
//绘制干扰线
$line=5;
if($line){
  for($i=1;$i<$line;$i++){
    $color = imagecolorallocate ( $image, mt_rand ( 50, 90 ), mt_rand ( 80, 200 ), mt_rand ( 90, 180 ) );
    imageline($image, mt_rand(0,$width-1), mt_rand(0,$height-1), mt_rand(0,$width-1), mt_rand(0,$height-1),$color);
  }
}
//规定输出格式
header ( "content-type:image/gif" );
//建立gif图形
imagegif ( $image );
//销毁图像
imagedestroy ( $image );
```

### 3.2 检查图像

在浏览器访问/lib/image.func.php , 观察图像验证码是否显示成功。

> 注意：如果图形验证码无法显示，优先检查字体文件是否存放正确，文件名是否正确

### 3.3 封装方法

添加可选参数，将生成验证码图像程序封装成 **verifyImage()** 方法：

```php
[PHP]
function verifyImage($type=1, $length=4, $pixel=30, $line=5, $sess_name="verify"){
  //启用session
  session_start();
  //创建画布
  $width = 80;
  $height = 28;
//建立宽为width，高为height的画布
  $image = imagecreatetruecolor ( $width, $height );
//设定一些颜色
  $white = imagecolorallocate ( $image, 255, 255, 255 );
  $black = imagecolorallocate ( $image, 0, 0, 0 );
//用填充矩形填充画布
  imagefilledrectangle ( $image, 1, 1, $width - 2, $height - 2, $white );
//调用创建字符串方法并传递$type,$length参数
  $chars = buildRandomString ( $type, $length );
//把字符串存入SESSION
  $_SESSION [$sess_name] = $chars;
//设置一些字体
  $fontfiles = array ("STXINGKA.TTF" );
//产生长度为length的字符图像
  for($i = 0; $i < $length; $i ++) {
    $size = mt_rand ( 14, 18 );
    //角度-15到15
    $angle = mt_rand ( - 15, 15 );
    //字符显示的位置
    $x = 5 + $i * $size;
    $y = mt_rand ( 20, 26 );
    //从设置的字体文件中随机取一个字体
    $fontfile = "../fonts/" . $fontfiles [mt_rand ( 0, count ( $fontfiles ) - 1 )];
    //随机取一个颜色
    $color = imagecolorallocate ( $image, mt_rand ( 50, 90 ), mt_rand ( 80, 200 ), mt_rand ( 90, 180 ) );
    //截取，从i开始取，取1位
    $text = substr ( $chars, $i, 1 );
    //将TTF文字写到图中
    imagettftext ( $image, $size, $angle, $x, $y, $color, $fontfile, $text );
  }
  if($pixel){
    for($i=0;$i<$pixel;$i++){
      imagesetpixel($image, mt_rand(0, $width-1),mt_rand(0, $height-1),$black);
    }
  }
  if($line){
    for($i=1;$i<$line;$i++){
      $color = imagecolorallocate ( $image, mt_rand ( 50, 90 ), mt_rand ( 80, 200 ), mt_rand ( 90, 180 ) );
      imageline($image, mt_rand(0,$width-1), mt_rand(0,$height-1), mt_rand(0,$width-1), mt_rand(0,$height-1),$color);
    }
  }
//规定输出格式
  header ( "content-type:image/gif" );
//建立gif图形
  imagegif ( $image );
//销毁图像
  imagedestroy ( $image );
}
```

> 注意：`session_start();` 在一个页面中只能调用一次，若页面中有两个或以上开启session的代码，应当只保留一个，否则会出错，在引入其他页面时需要注意。

### 3.4 调用验证码方法

在 **getVerify.php** 中调用验证码方法：

```php
[PHP]
<?php
require_once '../lib/image.func.php';
//如写include文件，只需require_once '../include.php'，下同
verifyImage();
```

浏览器访问/admin/getVerify.php 检查是否成功生成验证码。

> 注意：verifyImage() 中的参数可选

include文件参考：

```php
[PHP]
define("ROOT",dirname(__FILE__));
set_include_path(".".PATH_SEPARATOR.ROOT."/lib".PATH_SEPARATOR.ROOT."/core".PATH_SEPARATOR.get_include_path());
require_once 'image.func.php';
require_once 'string.func.php';
```



## 4. 前台显示

在 **login.php** 中显示生成的验证码：

```HTML
[HTML]
<ul class="login">
  <li class="l_tit">验证码</li>
  <img src="getVerify.php" alt="verify"/>
  <li class=""><input type="text" class="login_input user_icon"></li>
  <li><input type="button" value="" class="login_btn">登录</li>
```

浏览器访问 /admin/login.php 检查是否显示成功

## 5. 验证

验证相关的内容在表单验证的文档中介绍。



> May 11, 2016 BY 天真