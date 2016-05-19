---
title: PHP分页制作方法
date: 2016-05-16 15:56:22
tags: 
      - 后台
      - php
      - 技术
toc: true
---

# PHP分页制作方法

本文介绍了用PHP制作分页的方法。

[TOC]

## 1. 准备工作

### 1.1 数据库

在数据库中导入一系列数据，本文所用的表表名为 **demo_page**，结构如下：

| id   | name | age  |
| ---- | ---- | ---- |
| 1    | 张三   | 16   |

## 2. 分页方法

在 **page.func.php** 中写分页方法：

<!--more-->

```PHP
[PHP]
//引入一些连接数据库之类的文件
require_once '../include.php';
//得到记录的总条数
$sql="select * from demo_page";
$result=mysql_query($sql);
$totalRows=mysql_numrows($result);
//echo $totalRows;  验证是否成功取得记录条数
//每页显示的记录条数
$pageSize=20;
//得到总页数
$totalPages=ceil($totalRows/$pageSize);
//分页
$page=$_REQUEST['page']?(int)$_REQUEST['page']:1;
if($page<1||$page==null||!is_numeric($page)){
  $page=1;
}
if($page>=$totalPages){
  $page=$totalPages;
}
$offset=($page-1)*$pageSize;
$sql="select * from demo_page limit {$offset},{$pageSize}";
$rows=fetchAll($sql);
//print_r($rows);  验证是否能够正确取值
//输出取得的信息
foreach($rows as $row){
  echo "id:".$row['id'],"</br>";
  echo "name:".$row['name'],"</br>";
  echo "age:".$row['age'],"</br>";
  echo "<hr>";
}
//建立跳转链接，生成页码
$url=$_SERVER['PHP_SELF'];
$index=($page==1)?"首页":"<a href='{$url}?page=1'>首页</a>";
$prev=($page==1)?"":"<a href='{$url}?page=".($page-1)."'>上一页</a>";
$next=($page==$totalPages)?"":"<a href='{$url}?page=".($page+1)."'>下一页</a>";
$last=($page==$totalPages)?"尾页":"<a href='{$url}?page={$totalPages}'>尾页</a>";
$str="共{$totalPages}页/当前第{$page}页";
for($i=1;$i<=$totalPages;$i++){
  //当前页
  if($page==$i){
    $p.="[{$i}]&nbsp";
  }else{
    $p.="<a href='{$url}?page={$i}'>[{$i}]</a>&nbsp";
  }
}
echo "<hr>";
echo $index."&nbsp".$prev."&nbsp".$p.$next."&nbsp".$last."</br>".$str;
```

## 3. 封装方法

将过程封装为方法以便使用：

```php
[PHP]
function showPage($page, $totalPages, $where=null)
{
  //$where的作用是假设传值有其他条件，则自动在url上添加
  $where=($where==null)?null:"&".$where;
  $url = $_SERVER['PHP_SELF'];
  $index = ($page == 1) ? "首页" : "<a href='{$url}?page=1'>首页</a>";
  $prev = ($page == 1) ? "" : "<a href='{$url}?page=" . ($page - 1) . "'>上一页</a>";
  $next = ($page == $totalPages) ? "" : "<a href='{$url}?page=" . ($page + 1) . "'>下一页</a>";
  $last = ($page == $totalPages) ? "尾页" : "<a href='{$url}?page={$totalPages}'>尾页</a>";
  $str = "共{$totalPages}页/当前第{$page}页";
  for ($i = 1; $i <= $totalPages; $i++) {
    if ($page == $i) {
      //当前页不显示链接
      //@的作用是屏蔽错误信息
      @$p .= "[{$i}]&nbsp";
    } else {
      @$p .= "<a href='{$url}?page={$i}'>[{$i}]</a>&nbsp";
    }
  }
  $pageStr=$index . "&nbsp" . $prev . "&nbsp" . $p . $next . "&nbsp" . $last . "</br>" . $str;
  return $pageStr;
}
//调用方法以验证结果
echo showPage($page, $totalPages);
```

以上

> May  16, 2016 By 天真