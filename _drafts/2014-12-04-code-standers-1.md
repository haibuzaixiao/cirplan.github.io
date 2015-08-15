---
layout: post
category : 编码规范
title: 前端编码规范（1）-- html
tags : [html, css, js]
---
{% include JB/setup %}

搞前端也挺久了，但代码一直没有规范的写，都是随心所欲，爱咋咋地。最近QL师兄丢了网站过来，真心给力，前端的都可以去看看。[isobar编码规范][1]。下面把一些我认为比较重要的copy出来。

## **前端开发核心思想**

1.  表现、内容和行为的分离。
2.  标记应该是结构良好、语义正确 以及 普遍合法 。
3.  `Javascript`应该起到渐进式增强用户体验的作用 。

## **总体原则**

###**缩进**

对于所有编程语言，要求缩进必须是软`tab`（用空格字符）。在文本编辑器里敲 `Tab` 应该等于 4个空格 。

###**可读性 vs 压缩**

对于维护现有文件，认为可读性比节省文件大小更重要。大量空白和适当的`ASCII`艺术都是受鼓励的。任何开发者都不必故意去压缩`HTML`或`CSS`，也不必把`Javascript`代码最小化得面目全非。

会在服务器端或`build`过程中自动最小化并`gzip`压缩所有的静态客户端文件，例如`CSS`和`JS`。

## **HTML**

###**Doctype**

使用合适的`Doctype`来指示浏览器触发标准模式. 永远要避免`Quirks`模式。

	<!DOCTYPE html>

###**字符编码**

    <meta charset="utf-8">

###**属性加引号**

在`HTML5`规范里并没有严格要求属性值两边加引号。但考虑到一些属性可以接受空白值，为了保持一致性，所有属性值必须加上引号。

###**标签的总体原则**

在创建的`HTML`文档里总是要使用能够代表内容语义的标签。

*  段落分隔符要使用实际对应的`<p>`元素，而不是用多个`<br>`标签。
*  在合适的条件下，充分利用`<dl>` （定义列表）和`<blockquote>` 标签。
*  列表中的条目必须总是放置于`<ul>`、`<ol>`或`<dl>` 中，永远不要用一组 `<div>`或`<p>` 来表示。
*  给每个表单里的字段加上 `<label>` 标签，其中的 `for`属性必须和对应的输入字段对应，这样用户就可以点击标签。同理，给标签加上 `cursor:pointer`; 样式也是明智的做法。
*  在某些闭合的 `</div>`标签旁边加上一段`html`注释，说明这里闭合的是什么元素。这在有大量嵌套和缩进的情况下会很有用。
*  不要把表格用于页面布局。
*  在合适的条件下，利用 `<thead>`、`<tbody>`和`<th>`标签.

下面贴上[html5-boilerplate][2]的标准首页。

	<!DOCTYPE html>
	<!--[if lt IE 7]>      <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
	<!--[if IE 7]>         <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
	<!--[if IE 8]>         <html class="no-js lt-ie9"> <![endif]-->
	<!--[if gt IE 8]><!--> <html class="no-js"> <!--<![endif]-->
	    <head>
	        <meta charset="utf-8">
	        <meta http-equiv="X-UA-Compatible" content="IE=edge">
	        <title></title>
	        <meta name="description" content="">
	        <meta name="viewport" content="width=device-width, initial-scale=1">

	        <!-- Place favicon.ico and apple-touch-icon.png in the root directory -->

	        <link rel="stylesheet" href="css/normalize.css">
	        <link rel="stylesheet" href="css/main.css">
	        <script src="js/vendor/modernizr-2.6.2.min.js"></script>
	    </head>
	    <body>
	        <!--[if lt IE 7]>
	            <p class="browsehappy">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
	        <![endif]-->

	        <!-- Add your site or application content here -->
	        <p>Hello world! This is HTML5 Boilerplate.</p>

	        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
	        <script>window.jQuery || document.write('<script src="js/vendor/jquery-1.10.2.min.js"><\/script>')</script>
	        <script src="js/plugins.js"></script>
	        <script src="js/main.js"></script>

	        <!-- Google Analytics: change UA-XXXXX-X to be your site's ID. -->
	        <script>
	            (function(b,o,i,l,e,r){b.GoogleAnalyticsObject=l;b[l]||(b[l]=
	            function(){(b[l].q=b[l].q||[]).push(arguments)});b[l].l=+new Date;
	            e=o.createElement(i);r=o.getElementsByTagName(i)[0];
	            e.src='//www.google-analytics.com/analytics.js';
	            r.parentNode.insertBefore(e,r)}(window,document,'script','ga'));
	            ga('create','UA-XXXXX-X');ga('send','pageview');
	        </script>
	    </body>
	</html>

这样就可以根据自己的需要去更改了。当然，前提你得先有一份boilerplate的源码。

总是使用能够代表内容语义的标签是比较重要的，从以前的`table`布局到现在`div`+`css`布局，这是始终如一的，不能盲目的堆叠`div`之类的标签。

[1]: http://coderlmn.github.io/code-standards/
[2]: http://html5boilerplate.com
