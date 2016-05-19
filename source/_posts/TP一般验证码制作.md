---
title: TP一般验证码制作
date: 2016-04-24 15:56:22
tags: 
      - 后台
      - php
      - thinkphp
      - 技术
toc: true
---

# 使用Thinkphp3.2.3框架制作简单的验证码

[TOC]
## 1. 实例化

实例化生成验证码的类(写入控制器)

```php
[PHP]
public function verify_c(){
	$Verify = new \Think\Verify();
	$Verify->fontSize = 18;
	$Verify->length = 4;

	$Verify->useNoise = false;
	$Verify->codeSet = '0123456789';
	$Verify->imageW = 130;
	$Verify->imageH = 50;
	//$Verify->expire = 600;
	$Verify->entry();
}  
```
## 2. 调用图片

在前台生成验证码的图片src属性指向

<!--more-->

```html
[HTML]
<form action="{:U('Verify/pv',array())}" method="post" id="captcha-container">
	<input name="code" width="50%" height="50" class="captcha-text" 
	placeholder="验证码" type="text" >
	<img width="130" class="left15" height="35" alt="验证码" 
	src="{:U('Home/Verify/verify_c',array())}" title="点击刷新">
	<input type="submit" value="提交"/>
</form>
```
## 3. 点击刷新

用jquery实现点击刷新验证码

```javascript
[JavaScript]
//验证码生成
var captcha_img = $('#captcha-container').find('img')  
var verifyimg = captcha_img.attr("src");  
captcha_img.attr('title', '点击刷新');  
captcha_img.click(function(){  
	if( verifyimg.indexOf('?')>0){  
		$(this).attr("src", verifyimg+'&random='+Math.random());  
	}else{  
		$(this).attr("src", verifyimg.replace(/\?.*$/,'')+'?'+Math.random());  
	}  
}); 
```

如果从外部引入js文件，引入的文件需在jquery之后，如不封装方法，引入的文件需在html元素之后

## 4. 校验验证码
在控制器中检查验证码是否正确

```php
[PHP]
/* 验证码校验 */
public function pv()
{
	if (!empty($_POST)) {
		$verify = new \Think\Verify();
		$code = $_POST['code'];
		if(!$verify->check($code))
		{
			$this->error("验证码错误！");
			return;
		} else {
			echo "验证码正确！！";
			}
	}
}
```

> April 24, 2016   By 天真小兮兮
