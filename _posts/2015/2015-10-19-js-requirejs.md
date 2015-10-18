---
layout: post
category : Js
title: JS模块化开发（3）：require.js的使用以及压缩策略
tags : [js, requirejs, module]
---
{% include JB/setup %}

[require.js][1]是遵循AMD规范的模块加载器，这里简单说说其用法和压缩策略。

##本文内容：

1.基本使用

2.如何打包压缩

##基本使用

###1.1 加载

要使用require.js前提当然是先下载好或引入CDN。若下载好后，直接引入：

	<script data-main="index" src="/js/require.js"></script>

这里表示引入require.js后，把index.js作为程序的主模块，加载并执行。如果怕页面阻塞，可以把这段代码放到页面底部。

###1.2 定义主模块

定义主模块可以使用`require`：

	// main.js
	require(['moduleA', 'moduleB', 'moduleC'], function (moduleA, moduleB, moduleC){
	    // some code here
	});

也可以使用常规的定义模块的方式，用`define`定义，因为`data-main`已经指定这个是主模块了：

	// main.js
	define(['moduleA', 'moduleB', 'moduleC'], function (moduleA, moduleB, moduleC){
	    // some code here
	});

但有一种情况是一定要使用`require`定义的，下面会说到。

###1.3 require.js配置

利用`require.config()`，对require.js进行基本配置，`require.config()`要放在主模块的头部，主要作用如下：

* 定义基本目录；
* 对常用模块定义别名，方便引用；
* 定义文件版本号；
* 处理非AMD规范的模块；
* 处理非AMD规范并对其他模块产生依赖的模块；

下面我们看例子：

	// require.js config
	require.config({
	    baseUrl: '/common/js', // 定义基本目录
	    urlArgs: 'v=1.1.0',  // 定义请求js文件版本号
	    paths: { // 常用模块别名
	        'angular'    : 'lib/angular.min',
	        'sanitize'   : 'lib/angular-sanitize.min',
	        'zepto'      : 'lib/zepto.min'
	    },

	    // 若加载模块不符合AMD规范，直接返回全局变量，则用下面方式封装
	    // 切记：exports的变量一定要和返回的 全局变量名称 一致
	    shim: {
	        'angular': {
		        exports: 'angular'
	        },
	        'sanitize': { // 对非AMD规范，对其他模块产生依赖依赖的要加上
		        deps: ['angular'],
		        exports: 'Sanitize'
	         },
	        'zepto': {
		       exports: 'Zepto'
	       }
	    }

	});

上面这个就是一个很常规的require.config配置。但在实际开发中，我们是不会把这段配置加在每个主模块的头部的，不然要修改点东西，就要一个一个改，岂不是要死的心都有。

一般把上面的配置信息提成一个公共js，在require.js加载前加载。

	<script src="/js/config.js"></script>
	<script data-main="index/index" src="/js/require.js"></script>

但如果直接把上面这段配置放在`config.js`里会报`require is undefined`。想想也是，require.js都还没有加载。所以我们修改下：

	// require.js config
	var require = {
	    baseUrl: '/common/js', // 定义基本目录
	    urlArgs: 'v=1.1.0',  // 定义请求js文件版本号
	    paths: { // 常用模块别名
	        'angular'    : 'lib/angular.min',
	        'sanitize'   : 'lib/angular-sanitize.min',
	        'zepto'      : 'lib/zepto.min'
	    },

	    // 若加载模块不符合AMD规范，直接返回全局变量，则用下面方式封装
	    // 切记：exports的变量一定要和返回的 全局变量名称 一致
	    shim: {
	        'angular': {
		        exports: 'angular'
	        },
	        'sanitize': { // 对非AMD规范，对其他模块产生依赖依赖的要加上
		        deps: ['angular'],
		        exports: 'Sanitize'
	         },
	        'zepto': {
		       exports: 'Zepto'
	       }
	    }

	});

这样就ok啦，现在可以想怎么折腾就怎么折腾了。

###1.4 AMD模块写法

关于AMD模块的定义，上一篇有介绍。直接定义在`define`函数中。如我们要定义url模块：

	// url.js
	define(function (){
	　　var getId = function (key) {
		    ...
	    };

	　　return {
	       getId: getId
	　　};
	});

使用方法如下：

	// main.js
	require(['url'], function (url){
	　　alert(url.getId('index'));
	});

若定义有依赖的模块：

	define(['util'], function(util){
	    function each(){
	        util.doSomething();
	        ...
	    }
	    return {
	       each : each
	    };
	});

##2.如何打包压缩

由于使用模块化，文件会被分成很多个，这样在浏览器端无疑会产生很多HTTP请求，会让用户的等待时间加长。对于这个问题，require.js也提出了用`r.js`合并压缩的方案。

未完待续...


[1]: http://requirejs.org/


