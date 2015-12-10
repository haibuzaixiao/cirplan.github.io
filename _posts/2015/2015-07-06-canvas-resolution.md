---
layout: post
category : JS
title: canvas分辨率问题
tags : [js, canvas]
---
{% include JB/setup %}

过程：最近尝试用H5的canvas写了个小游戏，然后加载图片的时候发现，在电脑上图片是很清晰的，但到了手机上就看到比较模糊（图片都是实际大小的2倍）。

再现：demo代码如下

	window.onload = function () {
		var canvas = document.getElementById('canvas1');
		var ctx = canvas.getContext("2d");
		var width, height;
		width = 320;
		height = 568;

		canvas.style.width = width+'px';
		canvas.style.height = height + 'px';
		canvas.width = width; 
		canvas.height = height;
		var img = new Image();
		img.src = "../img/s6_1.png";
		img.onload = function () {  
		    ctx.drawImage(img, 20, 20, img.width/2, img.height/2); 
		}  
	}();

先看效果图（左边为电脑模拟，右边为手机）：

![电脑](/images/2015/20150706canvas/1_pc.png)
![手机](/images/2015/20150706canvas/2_phone.jpg)

有没有发现什么？认真看就会发现：右边的图比较模糊（如果实在没发现请自戳双目，为什么？因为留着没用）。

所以现在问题是：`为什么canvas加载图片在手机上会显示模糊？`

首先我们看看上面的代码：

	canvas.style.width = width+'px';
	canvas.style.height = height + 'px';
	canvas.width = width; 
	canvas.height = height;

这样子写是不是有点多此一举了？闲的蛋疼？为了证明不是蛋疼，首先我们要了解`设备像素`和`CSS像素`。

### 设备像素和CSS像素

设备像素就是我们平时所说的设备真实像素，像iPhone5s的 640 x 1136。但一般开发者是不怎么关心这个数值的，因为有CSS像素。

CSS像素是我们平时代码直接用的，iPhone5s的CSS像素为 320 x 568（这样子应该懂了吧）。

由于设备像素和CSS像素不一样，所以设备会用几个设备像素表示一个CSS像素。像iPhone5s就是4个设备像素点表示一个CSS像素。如图：

![像素比例](/images/2015/20150706canvas/3_px.gif)

这里就有一个很常见的问题：为什么图片在retina屏中会模糊？

例如要显示 100px x 100px 的图片，如果切出来的图片也是 100px x 100px，那么在iPhone5s下是肯定会模糊的。
因为iPhone5s的 设备像素/CSS像素 比是2。所以它需要的图片实际大小为 200px x 200px。如果像现在这样大小不够呢？那像素点只能被扩充了。

什么是扩充？就是5s的4个设备像素点表示1个CSS像素点时候，本应该4个像素点都有各自的颜色的，但图片小了，那4个设备像素点都是复制成同一种颜色。十分简单粗暴的方法。

`canvas的显示也是类似于图片，如果不够大就会被扩充。` 

`canvas.width/height` 表示的是设备像素，而 `canvas.style.width/height` 表示的是CSS像素。所以设置`canvas.width/height==canvas.style.width/height`的时候，如果 设备像素/CSS像素 不是1，canvas就被扩充了，然后图片加载进去，虽然是2倍实际大小，但还是模糊掉了（这里没想通？再想想）。这样就很好解析刚刚的例子在电脑端没有模糊（我的电脑像素比是1），但在手机端却模糊了（还不明白？不是我的问题就是你的问题了）。

### 解决方法

既然问题出现了，总是会有解决方法的。先看两个参数 `devicePixelRatio`和`webkitBackingStorePixelRatio`。

#### devicePixelRatio

devicePixelRatio是window下的属性，是 设备像素/CSS像素 的比值。下面列出了一些：

* iPhone5s 		2
* iPhone6 plus  3
* nexus7 		1.3 (非常奇葩)

这个参数比较简单直观，不用多说。其实在chrome模拟手机浏览的时候，也有这个参数的，如图：

![chrome模拟像素比](/images/2015/20150706canvas/4_pc_px.png)

上图中红色框中的就是这个像素比。

#### webkitBackingStorePixelRatio

webkitBackingStorePixelRatio是存在canvas context中（仅safari和chrome），该属性的值决定了浏览器在渲染canvas之前会用几个像素来存储画布信息。
例如，在iOS6 safari中这个值为2，它要渲染 100px x 100px 的图片，首先会在内存中生成一张200x200的图片，然后浏览器渲染的时候，会按100x100的图片来渲染，
因此就变成了200x200，正好和内存中的图片大小一致，因此在iOS的safari中不会出现失真的问题（有点绕口，慢慢理解）。

但是在iOS7 的safari中这个值又变成了1，是出于性能的考虑，详情可以看[这里][1]，搜索关键字‘backing’。

知道了这俩个参数，那接下来怎么玩？实践是检验真理的唯一标准。

### 实践

修改之前的代码，把像素比加进去：

	var PIXEL_RATIO = (function () {
	    var ctx = document.createElement("canvas").getContext("2d"),
	        dpr = window.devicePixelRatio || 1,
	        bsr = ctx.webkitBackingStorePixelRatio ||
	              ctx.mozBackingStorePixelRatio ||
	              ctx.msBackingStorePixelRatio ||
	              ctx.oBackingStorePixelRatio ||
	              ctx.backingStorePixelRatio || 1;

	    return dpr / bsr;
	})();

	window.onload = function () {
		var canvas = document.getElementById('canvas1');
		var ctx = canvas.getContext("2d");
		var width, height;
		width = 320;
		height = 568;

		canvas.style.width = width+'px';
		canvas.style.height = height + 'px';
		canvas.width = width * PIXEL_RATIO; 
		canvas.height = height * PIXEL_RATIO;
		var img = new Image();
		img.src = "../img/s6_1.png";
		img.onload = function () {  
		    ctx.drawImage(img, 20, 20, img.width/2, img.height/2); 
		}  
	}();

看看效果（左边为之前效果，右边为修改后效果）：

![之前](/images/2015/20150706canvas/1_pc.png)
![之后](/images/2015/20150706canvas/5_pc.png)

发现了什么？图居然缩小了！我滴天，这是为什么？认真想想问题出在哪里。

`因为CSS像素点被缩放了`，为什么会缩放？因为它是大爷，它想缩所以缩了。

所以解决方案是对canvas进行缩放，下面是完整版代码：

	var PIXEL_RATIO = (function () {
		    var ctx = document.createElement("canvas").getContext("2d"),
		        dpr = window.devicePixelRatio || 1,
		        bsr = ctx.webkitBackingStorePixelRatio ||
		              ctx.mozBackingStorePixelRatio ||
		              ctx.msBackingStorePixelRatio ||
		              ctx.oBackingStorePixelRatio ||
		              ctx.backingStorePixelRatio || 1;

		    return dpr / bsr;
	})();

	window.onload = function () {
		var canvas = document.getElementById('canvas1');
		var ctx = canvas.getContext("2d");
		var width, height;
		width = 320;
		height = 568;

		canvas.style.width = width+'px';
		canvas.style.height = height + 'px';
		canvas.width = width * PIXEL_RATIO; 
		canvas.height = height * PIXEL_RATIO;

		ctx.scale(PIXEL_RATIO, PIXEL_RATIO);

		var img = new Image();
		img.src = "../img/s6_1.png";
		img.onload = function () {  
		    ctx.drawImage(img, 20, 20); 
		}  
	}();

左边是电脑之前模拟的效果，右边是电脑现在模拟的效果：

![之前电脑](/images/2015/20150706canvas/1_pc.png)
![之后电脑](/images/2015/20150706canvas/7_pc.png)

左边是手机之前的效果，右边是手机现在的效果：

![之前手机](/images/2015/20150706canvas/2_phone.jpg)
![之后手机](/images/2015/20150706canvas/8_ph.jpg)

嗯，基本的解决方案就是这样。如果有问题，可以提出来～

参考：

* [http://blog.csdn.net/laijingyao881201/article/details/39505043][2]
* [http://www.html5rocks.com/en/tutorials/canvas/hidpi/][3]

[1]:http://asciiwwdc.com/2013/sessions/600
[2]:http://blog.csdn.net/laijingyao881201/article/details/39505043
[3]:http://www.html5rocks.com/en/tutorials/canvas/hidpi/










