---
title: Jquery-validate(表单验证)插件的用法
date: 2016-04-26 15:56:22
tags: 
      - JavaScript
      - 前端
      - 技术
toc: true
---
# Jquery-validate(表单验证)插件的用法


Jquery-validate官网地址：[https://jqueryvalidation.org/](https://jqueryvalidation.org/)

示例版本：jquery-validate-1.13.1

[TOC]

## 1. 引入插件
在html文件中引入jquery框架和jquery-validate插件

```html
[html]	
<head>
  <meta charset="UTF-8">
  <title>表单验证</title>
  <script src="__PUBLIC__/js/jquery-1.9.0.min.js"type="text/javascript" charset="utf-8"></script>
  <script src="__PUBLIC__/js/jquery.validate-1.13.1.js" type="text/javascript" charset="utf-8">
  </script>
</head>
```

## 2. 使用默认验证

html代码如下：

```html
[html]
<form action="" method="post" name="register" id="register">
	<span id="">
		<input type="text" name="username" id="username" value="" placeholder="name"/>
	</span>
    <span id="">
		<input type="password" name="password" id="password" placeholder="password" />
	</span>
    <input type="submit" value="OK" />
</form>
```

<!--more-->

js验证代码如下：

```javascript
[Javascript]
<script type="text/javascript">
      $(document).ready(function(){
        $("#register").validate({
          rules:{
            username: {
              required: true,
              minlength:2,
              maxlength:6
            },
            password: {
              required:true,
              minlength:3,
              maxlength:8
            },
          },
          messages:{
            username: {
              required: "用户名必须填写",
              minlength: "用户名最小为两位",
              maxlength: "用户名最大为六位"
            },
            password: {
              required:"密码必须填写",
              minlength:"密码最小为三位",
              maxlength:"密码最大为十位"
            },
          }
        });
      });
</script>
```

> rules中的“username”， “password”是name属性的值，messages是验证失败的错误提示，可根据实际情况修改。

## 3. remote方法

用于远程校验，例：

```javascript
[Javascript]
rules:{
  username: {
  required: true,
  minlength:2,
  maxlength:6,
  remote:"remote.json" 
 },
```

则提交后，会向"remote.json"提交一个get请求

### remote的配置

```javascript
[Javascript]
remote:{
  url:"remote.json",
  type:"post",//设置请求方式为post（默认为get）
  data:{
    loginTime:function(){
      return + new Date;//返回当前时间
    }
  }
}
```

## 4. 基本验证方法

- **required**---必填

- **remote**---远程校验
- **minlength**---最小长度，**maxlength**---最大长度， **rangelength**---长度范围--[2, 10]
- **min**---最小值， **max**---最大值， **range**---取值范围--[2, 10]
- **email**---email格式， **url**---url格式(*必须带有http或https*)
- **date**---日期格式（*日月年格式无效*）， **dateISO**---ISO日期（*格式为：年月日*）
- **number**---数字， **digits**---整数， **equalTo**---与另一个元素值相等（*使用选择器*）

## 5. 高级API 

### 5.1 valid方法：

  用于检查表单是否有效，返回bool值，例：

```javascript
  [Javascript]
  $("#check").click(function(){
    alert($("#register").valid()?"正确":"错误")
  });
```

### 5.2 rules方法：

  rules（"add", rules）向表单元素增加校验规则

  rules（"remove",rules）删除表单元素校验规则

```javascript
  [Javascript]
  $("#username").rules("add",{minlength:2, maxlength:10})
```


### 5.3 validator对象方法

- validate方法会返回一个Validator对象，这个对象有很多方法：

  - **Validator.form()** 验证表单是否有效，返回bool
  - **Validator.element(element)** 验证某个元素是否有效，返回bool
  - **Validator.resetForm()** 把表单恢复到验证前原来的状态
  - **Validator.showErrors(errors)** 针对某个元素显示特定的错误信息
  - **Validator.numberOfInvalids()** 返回无效的元素数量


### 5.4 validator对象静态方法

- validator.format() 格式化字符串，例：

```javascript
  [Javascript]
  var template = $.validator.format("{0} - {1} - {2}");

  template("一", "二", "三")
```

  则输出为"一 - 二 - 三"。

- validator.setDefaults(options) 用于修改默认设置，例：

  ```javascript
  [Javascript]
  $.validator.setDefaults({
    debug:true
  });
  ```

  则所有的参数都有debug:true

- validator.addClassRules(name,rules) 为某些包含名为name的class增加组合验证类型

  ```javascript
  [Javascript]
  $.validator.addClassRules({
    txt:{
      required:true,
      minlength:5
    },
  });
  ```

## 6. validate()方法配置项

- **submitHandler** 通过验证后运行的函数，可以加上表单提交的方法

  ```javascript
  [Javascript]
  submitHandler:function(form){
    console.log($(form).serialize());//在控制台展示从表单中搜集到的信息
  }
  ```


- **invalidHandler** 无效表单提交后运行的函数

  ```javascript
  [Javascript]
  invalidHandler:function(event, validator){
    console.log("错误数：" + validator.numberOfInvalids);//在控制台展示错误数
  }
  ```

  该方法可以用以下方式触发：

  ```javascript
  [Javascript]
  $("#demoForm").on("invalid-form", function(event, validator){
    console.log("错误数：" + validator.numberOfInvalids);//在控制台展示错误数
  })
  ```

- **ignore** 对某些元素不进行验证

  ```javascript
  [Javascript]
  ignore: "#username"
  ```

  > ignore的默认值为":hidden"，即 `ignore` 默认不对隐藏的元素进行校验。

- **rules** 定义校验规则

    rule中可以使用depands来使某个设置在特定条件才生效，例：

  ```javascript
  [Javascript]
  rules:{
    username:{
      required:{
        depends:function(element){
          return $("#password").is(":filled");
        }
      },
    }
  }
  ```

  则在密码被填写时才校验


- **messages** 定义提示信息
  参照 `2.默认验证`
- **groups** 对一组元素的验证，用一个错误提示，用**errorPlacement** 控制把出错信息放在哪里

其他方法：

- **onsubmit** 是否在提交时验证


- **onfocusout** 是否在获取焦点时验证
- **onkeyup** 是否在敲击键盘时验证
- **onclick** 是否在鼠标点击时验证，一般用于`checkbox`或者`radio`
- **focusInvalid** 提交表单后，未通过验证的表单是否会获得焦点
- **focusCleanup** 当未通过验证的元素获得焦点时，是否移除错误提示
- **errorClass** 指定错误提示的css类名，可以自定义错误提示的样式
- **validClass** 指定验证通过的css类名
- **errorElement** 使用什么标签标记错误
- **wrapper** 使用什么标签把上边的errorElement包裹起来
- **errorLabelContainer** 把错误信息统一放在一个容器里
- **errorContainer** 显示或者隐藏验证信息，可以自动实现有错误信息出现时把容器属性变为显示，无错误时隐藏
- **showErrors** 可以显示总共有多少个未通过验证的元素
- **errorPlacement**  自定义错误信息放到哪里
- **success** 要验证的元素通过验证后执行的动作
- **highlight** 可以给未通过验证的元素加效果
- **unhighlight** 去除未通过验证的元素的效果，一般可以和highlight同时使用

## 7. 选择器扩展

- :**blank** 选择所有值为空的元素
- :**filled** 选择所有值不为空的元素
- :**unchecked** 选择所有没有被选中的元素

## 8. 自定义验证方法

- **jQuery.validator.addMethod( name, method [ , message ] )**

  name : 方法名称

  method : function(value, element, params) 方法逻辑

  message : 提示消息

  ```javascript
  [Javascript]
  $.validator.addMethod("postcode", function(value, element, parames){
    var postcode = /^[0-9]{6}$/;//0-9六位
    return this.optional(element) || (postcode.test(value));//符合则验证通过，前半段为允许空（空值也返回true）
  },"请填写正确的邮编！");
  ```

  ​


- **additonal-methods.js** 包含了许多扩展验证方法，使用时可参考

## 9. 安全性

不要相信任何客户端输入，任何提交必须在前端和服务器端做两次验证！



> April 26, 2016 By 天真小兮兮

