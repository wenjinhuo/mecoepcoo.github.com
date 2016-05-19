---
title: PHP实现文件上传与下载功能
date: 2016-05-19 14:00:02
tags:
      - php
      - 技术
      - 后台
toc: true
---

# PHP实现文件上传与下载功能

本文介绍用php实现文件上传和下载的方法。

[TOC]

## 1. 初步实现上传

### 1.1 客户端配置

建立 **upload.php** ：

```html
[html]
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>test</title>
</head>
<body>
  <!--此处只能使用post方法-->
  <form action="doUploadAction.php" method="post" enctype="multipart/form-data">
    <p>选择要上传的文件</p>
    <input type="file" name="uploadFile"/></br>
    <input type="submit" value="点击上传"/>
  </form>
</body>
</html>
```

<!--more-->

建立 **doUploadAction.php** ：

```php
header("content-type:text/html;charset=utf-8");
//$_FILES: 文件上传变量
$fileInfo=$_FILES['uploadFile'];
$filename=$fileInfo['name'];
$type=$fileInfo['type'];
$tmp_name=$fileInfo['tmp_name'];
$size=$fileInfo['size'];
$error=$fileInfo['error'];

if($error==UPLOAD_ERR_OK){
  //将临时文件移动到指定目录,并重命名
  if(move_uploaded_file($tmp_name, "upload/".$filename)){
    echo '文件'.$filename.'上传成功';
  }else{
    echo '上传失败';
  }
}else{
  //匹配错误信息
  switch($error){
    case 1:
      echo '文件大小超过了upload_max_filesize的最大值（默认2M）';
      break;
    case 2:
      echo '文件大小超过了html表单中设置的max_file_size的值';
      break;
    case 3:
      echo '文件没有上传完整';
      break;
    case 4:
      echo '没有文件被上传';
      break;
    case 6:
      echo '找不到临时文件夹';
      break;
    case 7:
      echo '文件写入失败';
      break;
    case 8:
      echo '上传的文件被php扩展程序中断';
      break;
  }
}
//以下方法也可实现,返回布尔值
//copy($tmp_name, "upload/".$filename);
```

>  出错调试：
>  使用 `print_r($FILES)` 输出 `$_FILES` 变量	:

```php

Array
(
    //“选择文件”按钮的name
    [uploadFile] => Array
        (
            //上传的文件名
            [name] => banner.jpg
            //上传文件的MIME类型
            [type] => image/jpeg
            //上传到服务器上的临时文件名
            [tmp_name] => C:\Users\Administrator\AppData\Local\Temp\phpF0EC.tmp
            //错误号
            [error] => 0
            //文件大小
            [size] => 131918
        )

)
```

**error** 报错信息：

- 0：上传成功
- 1：文件大小超过了upload_max_filesize的最大值（默认2M）
- 2：文件大小超过了html表单中设置的max_file_size的值
- 3：文件没有上传完整
- 4：没有文件被上传
- 6：找不到临时文件夹
- 7：文件写入失败
- 8：上传的文件被php扩展程序中断

客户端可使用的限制配置：

```html
<!--通过表单隐藏域限制上传文件的最大值-->
<input type='hidden' name ='MAX_FILE_SIZE' value= '字节数' />
```

```html
<!--通过accept属性限制上传文件类型-->
<input type='file' name ='uploadFile' accept= 'MIME类型' />
```

> <span style='color: #CC0033;font-weight: blod;'>注意：</span>客户端所做的限制并不安全，需要在服务器端再作限制

### 1.2 服务器端配置

服务器的配置文件在php安装目录下的 **php.ini** 中。

有关文件上传的配置：

- **file_uploads = On** ，支持http上传
- **upload_tmp_dir =**  ，临时文件的保存目录
- **upload_max_filesize = 2M** ，允许上传文件的最大大小
- **max_file_uploads = 20** ，每次允许上传的最大文件个数
- **post_max_size = 8M** ，post方式发送数据的最大值

扩展配置：

- **max_execution_time = -1** 设置脚本被解析器终止之前允许的最大执行时间，单位为秒
- **max_input_time = 60** 脚本解析输入数据允许的最大时间
- **memory_limit = 128M** 最大单线程的独立内存使用量



修改 **doUploadAction.php** ，做一些限制：

```php
header("content-type:text/html;charset=utf-8");
//$_FILES: 文件上传变量
$fileInfo=$_FILES['uploadFile'];
$maxSize=2097152;//允许的最大值，2M
$allowExt=array('jpg','jpeg','png','bmp','gif');//允许的扩展名
if($fileInfo['error']==0){
  //判断文件大小是否符合条件
  if($fileInfo['size']>$maxSize){
    exit('上传文件大小过大');
  }
  //允许的扩展名
  $ext=pathinfo($fileInfo['name'],PATHINFO_EXTENSION);
  //或者$ext=strtolower(end(explode('.',$fileInfo['name'])));
  //判断文件类型是否合法
  if(!in_array($ext,$allowExt)){
    exit('非法文件类型');
  }
  //判断是否通过http post方式上传
  //move_uploaded_file()函数只能对通过http post方式上传的文件有效
  //以下方法也可实现,返回布尔值
  //copy($tmp_name, "upload/".$filename);
  if(!is_uploaded_file($fileInfo['tmp_name'])){
    exit('文件不是通过http post上传');
  }
  //检测是否是真实的图片
  $flag=true;
  if($flag){
    if(!getimagesize($fileInfo['tmp_name'])){
      exit('不是真实的图片类型');
    }
  }
  $path='upload';
  //若没有路径则新建目录
  if(!file_exists($path)){
    mkdir($path,0777,true);
    chmod($path,0777);
  }
  //防止文件重名
  $uniName=md5(uniqid(microtime(true),true)).'.'.$ext;
  $destination=$path.'/'.$uniName;
  if(move_uploaded_file($fileInfo['tmp_name'],$destination)){
    echo '上传成功';
  }else{
    echo '上传失败';
  }
}else{
  //匹配错误信息
  switch($fileInfo['error']){
    case 1:
      echo '文件大小超过了upload_max_filesize的最大值（默认2M）';
      break;
    case 2:
      echo '文件大小超过了html表单中设置的max_file_size的值';
      break;
    case 3:
      echo '文件没有上传完整';
      break;
    case 4:
      echo '没有文件被上传';
      break;
    case 6:
      echo '找不到临时文件夹';
      break;
    case 7:
      echo '文件写入失败';
      break;
    case 8:
      echo '上传的文件被php扩展程序中断';
      break;
  }
}
```

> getimagesize()方法用以判断是否是真实的图片格式，真实图片的输出示例如下：

```php
//getimagesize($filename)  ，得到指定图片的信息，如果是图片则返回数组
//如果不是则返回false
$filename='banner.jpg';
var_dump(getimagesize($filename));

//返回示例：
//array(7) {
//  [0]=>
//  int(1200)
//  [1]=>
//  int(480)
//  [2]=>
//  int(2)
//  [3]=>
//  string(25) "width="1200" height="480""
//  ["bits"]=>
//  int(8)
//  ["channels"]=>
//  int(3)
//  ["mime"]=>
//  string(10) "image/jpeg"
//}
```

## 2. 上传函数的封装

编写 **upload.func.php** ：

```php
header("content-type:text/html;charset=utf-8");
$fileInfo=$_FILES['uploadFile'];
function uploadFile($fileInfo,$allowedExt=array('jpeg','jpg','png','gif'),
                    $maxSize=2097152,$path='upload',$flag=true){
  //判断错误号
  if($fileInfo['error']>0){
    //匹配错误信息
    switch($fileInfo['error']){
      case 1:
        $mes= '文件大小超过了upload_max_filesize的最大值（默认2M）';
        break;
      case 2:
        $mes= '文件大小超过了html表单中设置的max_file_size的值';
        break;
      case 3:
        $mes= '文件没有上传完整';
        break;
      case 4:
        $mes= '没有文件被上传';
        break;
      case 6:
        $mes= '找不到临时文件夹';
        break;
      case 7:
        $mes= '文件写入失败';
        break;
      case 8:
        $mes= '上传的文件被php扩展程序中断';
        break;
    }
    exit($mes);
  }
  $ext=pathinfo($fileInfo['name'],PATHINFO_EXTENSION);
  if(!is_array($allowedExt)){
    exit('扩展名应该是一个数组');
  }
  //检测上传文件的类型是否合法
  if(!in_array($ext,$allowedExt)){
    exit('非法文件类型');
  }
  //检测上传文件大小是否合法
  if($fileInfo['size']>$maxSize){
    exit('上传文件过大');
  }
  //判断是否通过http post方式上传
  //move_uploaded_file()函数只能对通过http post方式上传的文件有效
  //以下方法也可实现,返回布尔值
  //copy($tmp_name, "upload/".$filename);
  if(!is_uploaded_file($fileInfo['tmp_name'])){
    exit('文件不是通过http post上传');
  }
  //检测是否是真实的图片
  if($flag){
    if(!getimagesize($fileInfo['tmp_name'])){
      exit('不是真实的图片类型');
    }
  }
  //若没有路径则新建目录
  if(!file_exists($path)){
    mkdir($path,0777,true);
    chmod($path,0777);
  }
  //防止文件重名
  $uniName=md5(uniqid(microtime(true),true)).'.'.$ext;
  $destination=$path.'/'.$uniName;
  //移动文件
  if(!@move_uploaded_file($fileInfo['tmp_name'],$destination)){
    echo '上传失败';
  }
  return array(
    'newName'=>$destination,
    'type'=>$fileInfo['type'],
    'size'=>$fileInfo['size']
  );
}
```

方法的调用：

```php
include_once '../lib/upload.func.php';
$fileInfo=$_FILES['uploadFile'];
$newName=uploadFile($fileInfo);
print_r($newName);
```

