---
layout: post
category : JS
title: JS模块化开发3：require.js的使用以及压缩策略
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

	};

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

由于使用模块化，文件会被分成很多个，这样在浏览器端无疑会产生很多HTTP请求，会让用户的等待时间加长。对于这个问题，require.js也提出了用`r.js`合并压缩的方案。下面介绍r.js的使用。

使用r.js要安装node.js环境，未安装的自行安装。

###2.1 下载r.js

直接下载戳[这里][2]。或者直接用npm 安装

	npm install -g requirejs

###2.2 文件目录

下面是示例的test目录：

	test
	    -html
	        -index.html
	    -dist
	    -js
	        -index
	            -index.js
	            -index2.js
	        -lib
	            -require.js
	            -zepto.js
	        -util
	            -common.js
	            -url.js
	        -config.js
	    -r.js
	    -build.js

其中`r.js`放在根目录，`build.js`为压缩配置文件，`dist`为压缩后的输出目录。下面为对应js文件的内容：

	// index.js
	define(['zepto', 'index/index2', 'url'], function($, index2, url){
	    console.log('index');
	});

	// index2.js
	define(['url', 'util/common'], function(url, common){
	    console.log('index2');
	});

	// common.js
	define([], function(){
	    console.log('common');
	});

	// url.js
	define(['zepto'], function($){
	    console.log('url');
	});

	// config.js
	var require = {
	    baseUrl: "../js",
	    paths: {
	        "zepto": "lib/zepto",
	        "url": "util/url"
	    },
	    shim: {
	        'zepto':{
	        　　exports: 'Zepto'
	        }
	    }
	};

下面我们来看看如何压缩。

**需求1** 把index.js及其依赖项合并压缩成一个js

	({
	    baseUrl: './js',  // 设置基本目录
	    optimize: 'none', // 压缩方式：不压缩
	    // path和shim配置要和config.js一样
	    paths: { 
	　　　　　　"zepto": "lib/zepto",
	　　　　　　"url": "util/url"
	　　 },

	    shim: {
	        'zepto':{
	        　　exports: 'Zepto'
	        }
	    },

	    name: 'index/index', //设置要压缩的单个文件
	    out : 'js/index/index-build.js' // 设置要输出的文件名
	})	

在命令行进入到项目的根目录，执行：

	node r.js -o build.js

就可以看到在index目录下会生成新的文件，其中最后一部分为：

	...
	  ;['swipe', 'swipeLeft', 'swipeRight', 'swipeUp', 'swipeDown', 'doubleTap', 'tap', 'singleTap', 'longTap'].forEach(function(m){
	    $.fn[m] = function(callback){ return this.bind(m, callback) }
	  })
	})(Zepto)
	;
	define("zepto", (function (global) {
	    return function () {
	        var ret, fn;
	        return ret || global.Zepto;
	    };
	}(this)));

	define('url',['zepto'], function($){
		console.log('url');
	});
	define('util/common',[], function(){
		console.log('common');
	});
	define('index/index2',['url', 'util/common'], function(url, common){
		console.log('index2');
	});
	define('index/index',['zepto', 'index/index2', 'url'], function($, index2, url){
		console.log('index');
	});

**需求2** 把index.js和index2.js批量分别合并

因为在实际开发中很难像上面一个一个合并，所以批量合并是比较好得方法。修改build.js如下：

	({
	    baseUrl: './js',  // 设置基本目录
	    dir: './dist', // 设置输出目录
	    optimize: 'none', // 压缩方式：不压缩
	    // path和shim配置要和config.js一样
	    paths: { 
	　　　　　　"zepto": "lib/zepto",
	　　　　　　"url": "util/url"
	　　 },

	    shim: {
	        'zepto':{
	        　　exports: 'Zepto'
	        }
	    },

	    modules : [
	        {
	            name: 'index/index'
	        },

	        {
	            name: 'index/index2'
	        }
	    ]
	})

这样会在`dist`目录下生成一系列文件，如下图：

![](/images/2015/20151019requirejs/out1.png)

对于module数组里不要求合并的js，是和原来一样的。

调用直接调用这样就这样了，config.js都不用加载了：

	<script data-main="../dist/index/index.js" src="../js/lib/require.js"></script>

**需求3** 把zepto.js和url.js等公用文件合并成一个文件，index.js和index2.js如果有依赖这些文件时的，在合并的时候自动忽略

因为把所有页面的js都压缩成一个，公用的部分每次都加载，未免会增加等待时间，所以把公用部分抽出来是一个比较好的选择。

	({
	    baseUrl: './js',  // 设置基本目录
	    dir: './dist', // 设置输出目录
	    optimize: 'none', // 压缩方式：不压缩
	    // path和shim配置要和config.js一样
	    paths: { 
	　　　　　　"zepto": "lib/zepto",
	　　　　　　"url": "util/url"
	　　 },

	    shim: {
	        'zepto':{
	        　　exports: 'Zepto'
	        }
	    },

	    modules : [
	        {
	            name: 'lib/common',
	            create: true,
	            include: ['zepto', 'url']
	        },

	        {
	            name: 'index/index',
	            exclude: [
	                'zepto',
	                'url'
	            ]
	        },

	        {
	            name: 'index/index2',
	            exclude: [
	                'zepto',
	                'url'
	            ]
	        }
	    ]
	})

可以看到`dist/lib`下生成了`common.js`，而index.js变成了：

	// index.js
	define('util/common',[], function(){
		console.log('common');
	});
	define('index/index2',['url', 'util/common'], function(url, common){
		console.log('index2');
	});
	require(['zepto', 'index/index2', 'url'], function($, index2, url){
		console.log('index');
	});
	define("index/index", function(){});

修改`config.js`如下：

	var require = {
	    baseUrl: "../js",
	    paths: {
	        "zepto": "../dist/lib/common",
	        "url": "../dist/lib/common"
	    },

	    shim: {
	        'zepto':{
	        　　exports: 'Zepto'
	        }
	    }
	};

然后直接请求:

	<script src="../js/config.js"></script>
	<script data-main="../dist/index/index.js" src="../js/lib/require.js"></script>

这样就有公用模块了。更多r.js的配置看[这里][3]。

如果觉得requrejs压缩后还是很大，可以尝试使用require.js作者的另一个开源项目[almond.js][4]。almond.js压缩后最小为1k左右。
但有一个很重要的限制：所有模块必须压缩在一个文件里。almond.js的使用可以看[这里][5]。

最后，选择哪种压缩策略看项目的具体需要。

[1]: http://requirejs.org/
[2]: http://requirejs.org/docs/release/2.1.11/r.js
[3]: https://github.com/jrburke/r.js/blob/master/build/example.build.js
[4]: https://github.com/jrburke/almond
[5]: http://levi.yii.so/archives/3450

