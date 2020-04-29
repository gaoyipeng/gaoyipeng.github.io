---
title: Markdown基础语法
date: 2019-11-29 11:36:40
tags: Markdown
categories: 其他
description: MarkDown基础语法学习,开启个人博客时代
---
# MarkDown基础语法

## 标题

```	
用法:
# 一级标题
## 二级标题
```



## 段落及区块引用

```
用法:
> 高亮练习...
```

效果如下:
> 高亮练习...

<!--more-->

## 链接和图片

 ```
 用法:
 1、打开一个页面
 	<a href="https://www.cnblogs.com/" target="_blank">新页面跳转到博客园</a>
 2、当前页面打开
 	[当前页面跳转到博客园](https://www.cnblogs.com/")
 3、图片
 	![图片](https://upload-images.jianshu.io/upload_images/703764-605e3cc2ecb664f6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ```
 效果如下:

 <a href="https://www.cnblogs.com/" target="_blank">新页面跳转到博客园</a>

[当前页面跳转到博客园](https://www.cnblogs.com/)

![图片](https://upload-images.jianshu.io/upload_images/703764-605e3cc2ecb664f6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 列表

### 无序列表

```
用法:(*或+或-标识)
*	段落1

	>段落2沙发士大夫撒旦

	暗示法法师

*	段落2

	> sdfsasfa

* 	段落3
```

*	段落1

	>段落2沙发士大夫撒旦

	暗示法法师

*	段落2

	> sdfsasfa

* 	段落3

### 有序列表练习:
```
用法:
1. 1
2. 2
3. 3
```

1. 1
2. 2
3. 3

## 分割线

```
用法:
*** 
或者
---
```

效果如下:

---

## 强调

>使用\*或\_包裹即可。使用单一符号标记的效果是斜体，使用两个符号标记的效果是加粗

```
用法:

*这里是斜体*
_这里是斜体_

**这里是加粗**
__这里是加粗__
```
效果如下:

*这里是斜体*
_这里是斜体_

**这里是加粗**
__这里是加粗__

## 代码块

> 使用反引号 \`

一行代码:

`source.registerCorsConfiguration("/**", cors);`

多行代码:
```
	@Bean
    public CorsWebFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        CorsConfiguration cors = new CorsConfiguration();
        cors.setAllowCredentials(true);
        cors.addAllowedOrigin(CorsConfiguration.ALL);
        cors.addAllowedHeader(CorsConfiguration.ALL);
        cors.addAllowedMethod(CorsConfiguration.ALL);
        source.registerCorsConfiguration("/**", cors);
        return new CorsWebFilter(source);
    }
```
## 表格

```
用法

|序号(左对齐)|姓名(居中)|年龄(右对齐)|
|:---|:---:|---:|
|1   |张三  |23 |
|2   |李四  |33 |
```

>说明:默认居左,对中间使用几个-没有要求

:\-\-\- 表示文字居左

\-\-\-: 表示文字居右

:\-\-\-: 表示文字居中。

|序号(左对齐)|姓名(居中)|年龄(右对齐)|
|:---|:---:|---:|
|1   |张三  |23 |
|2   |李四  |33 |

## 特殊字符:使用时需要在前面加转义字符 \\

需要处理的特殊字符有:
```
\   反斜线
`   反引号
*   星号
_   底线
{}  花括号
[]  方括号
()  括弧
#   井字号
+   加号
-   减号
.   英文句点
!   惊叹号
```

例如:

\ 需要 写为 \\\\

\* 需要 写为 \\\*



> 参考:[https://www.jianshu.com/p/335db5716248](https://www.jianshu.com/p/335db5716248)

