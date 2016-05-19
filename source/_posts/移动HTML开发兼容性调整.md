---
title: 移动HTML开发兼容性调整(meta篇)
date: 2016-05-05 15:56:22
tags: 
      - HTML
      - 兼容性
      - 技术
toc: true
---

#  移动HTML开发兼容性调整(meta篇)

[TOC]

## 控制视口

在 `<head>` 标签中添加

```html
<meta name="viewpor" content="width=device-width, initial-scale=1, minimum-scale=1, maximun-scale=1, user-scalable=no">
```

达到控制视口和控制缩放的效果。

scale属性依次为：初始化缩放比例为1，最小缩放比例为1，最大缩放比例为1，是否允许用户缩放。

## 关闭手机号码检测

```html
<meta name="format-detection" content="telephone=no">
```

关闭邮箱检测为email

<!--more-->

## 强制渲染

以下代码可以强制使网页以webkit内核渲染：

```html
<meta http-equiv="X-UA-Compatible" content="chrome=1">
```

该命令不支持IE浏览器，可使用以下命令使IE用最高版本渲染：

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge, chrome=1">
```

## UC强制全屏

以下代码使UC浏览器强制全屏显示页面：

```html
<meta name="full-screen" content="yes">
```

## 禁止百度自动转码

以下代码禁止百度自动转码：

```html
<meta http-equiv="Cache-Control" content="no-siteapp" /> 
```

## 添加智能APP广告条

以下代码在safari浏览器上使用智能广告条：

```html
<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL"> 添加智能 App 广告条 Smart App Banner（iOS 6+ Safari）
```

## 极速模式

以下代码使浏览器强制使用极速模式（bootstrap官方文档有介绍）：

```html
<meta name="renderer" content="webkit">启用浏览器的极速模式(webkit)
```



> May 5, 2016 By *Mecoepcoo*

