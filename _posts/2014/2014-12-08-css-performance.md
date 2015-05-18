---
layout: post
category : css
title: 译.css性能
tags : [css, 翻译]
---
{% include JB/setup %}

原文：[CSS Performance][1]（2011年文章，有着很赞的翻转效果，前提是祭出高级浏览器）

**目录**

1. Reflows（重排）
2. Hardware Accelerated Css（硬件加速css）
3. Avoiding Expensive Css（避免高消耗css）
4. Selector Perf（选择器性能）
5. Style Matching Perf（样式匹配性能）
6. Polyfills
7. Debugging（调试）

## **1. 重排**
先来看看html页面的渲染、绘制流程图

![render](/images/render.png)

1. 根据`Html`标签生成`Dom`树（`Dom tree`）；根据默认的，用户自定义的样式生成样式结构（`Styles struct`）
2. 生成渲染树（`Render tree`）
3. 绘制页面

下面是重排的渲染短视频：

<video controls="" preload="" autobuffer="" width="860" height="510">
	<source src="https://dl.dropboxusercontent.com/u/39519/talks/cssperf/assets/geckoreflow-mozillaorg.mp4" type="video/mp4">
	<source src="https://dl.dropboxusercontent.com/u/39519/talks/cssperf/assets/geckoreflow-mozillaorg.webm" type="video/webm">
</video>

下面，先看看重绘机制：如何（不）触发`WebKit`的布局（[How (not) to trigger a layout in WebKit][2]）（2011）。

简单译文：

许多`Web`开发者意识到，一段`js`脚本在运行的时候，是在触发`DOM`的操作而不是执行代码本身。这样一个潜在消耗时间，从`Dom`树构建渲染树（`Render tree`）的过程，被称作布局（又称重排）。约庞大越复杂的`Dom`树，就会消耗更多的时间。

一个很重要的方法让页面的不假死，就是让分开的Dom操作放在一起。

ex: 

	// 一般，触发两次布局
	var newWidth = aDiv.offsetWidth + 10; // Read
	aDiv.style.width = newWidth + 'px'; // Write
	var newHeight = aDiv.offsetHeight + 10; // Read
	aDiv.style.height = newHeight + 'px'; // Write

	// 更好, 只触发一次布局
	var newWidth = aDiv.offsetWidth + 10; // Read
	var newHeight = aDiv.offsetHeight + 10; // Read
	aDiv.style.width = newWidth + 'px'; // Write
	aDiv.style.height = newHeight + 'px'; // Write

然后这里就会有个问题：什么情况会触发布局？下面是整理的一些元素和方法：

	Element
	clientHeight, clientLeft, clientTop, clientWidth, focus(), getBoundingClientRect(), getClientRects(), innerText, offsetHeight, offsetLeft, offsetParent, offsetTop, offsetWidth, outerText, scrollByLines(), scrollByPages(), scrollHeight, scrollIntoView(), scrollIntoViewIfNeeded(), scrollLeft, scrollTop, scrollWidth

	Frame, Image
	height, width

	Range
	getBoundingClientRect(), getClientRects()

	SVGLocatable
	computeCTM(), getBBox()

	SVGTextContent
	getCharNumAtPosition(), getComputedTextLength(), getEndPositionOfChar(), getExtentOfChar(), getNumberOfChars(), getRotationOfChar(), getStartPositionOfChar(), getSubStringLength(), selectSubString()

	SVGUse
	instanceRoot

	window
	getComputedStyle(), scrollBy(), scrollTo(), scrollX, scrollY, webkitConvertPointFromNodeToPage(), webkitConvertPointFromPageToNode()

当然列出的这些不是全部，这只是一个很好的开始。最好的方法是通过Chrome或者Firefox浏览器的调试器去查看。（翻译完）

好的，再回到这里，我们看看什么动作会触发重排。

* 增加，删除，更新`Dom`节点。
* 通过`display:none`隐藏元素。
* 对页面上的`Dom`元素进行移动等动画。
* 增加样式，或调整样式属性。
* 用户改变窗口大小，改变字体大小，或者滚动页面。

###策略：###
* 在触发重排动作前，对`Dom`进行批量操作。
* 复制节点的属性，在复制的节点里进行操作，在一次交换过来。
* 先用`display:none`隐藏节点，再进行大量的操作，再通过`display`显示。

更多重排文章：

[Stoyan Stefanov on Reflow/Repaint][3]

[The new game show: "Will it reflow?"][4]

[Mozilla's David Baron on Browser Internals for Web Developers][5]

[WebKit blog five-part series on rendering][6]

## **2.硬件加速css**##

充分利用`Css`的`transitions`和`transforms`达到最优的质量。
这方面主要应用于手机，ios和Android

<video controls="" preload="" autobuffer="" width="860" height="310" >
	<source src="https://dl.dropboxusercontent.com/u/39519/talks/cssperf/assets/translate3d.mp4" type="video/mp4">
	<source src="https://dl.dropboxusercontent.com/u/39519/talks/cssperf/assets/translate3d.webm" type="video/webm">
</video>

## **3.避免高消耗css**##

* @font-face 
* box-shadow 
* opacity compositing 
* gradients 
* text-align

(这部分待续...)

## **4.选择器性能**##

浏览器默认是从右到左。

下面选择器引擎是从左到右：

* Mootools
* Sly
* Peppy
* Dojo Acme
* Ext JS
* Prototype.js

下面是从右到左：

* Sizzle
* YUI 3
* NWMatcher
* querySelectorAll

选择器优化

推荐组合是：`tag .class`，让标签在左边，尽可能让class在右边。

## **5.样式匹配性能**##

关于样式匹配，请看[样式匹配性能][7]。

## **6.Polyfills**##

关于Polyfills的介绍：

>Polyfilling 是由 RemySharp 提出的一个术语，它是用来描述复制缺少的 API 和API 功能的行为。你可以使用它编写单独应用的代码而不用担心其他浏览器原生是不是支持。实际上，polyfills并不是新技术也不是和 HTML5 捆绑到一起的。我们已经在如json2.js，ie7-js 和为 IE 浏览器提供透明 PNG支持的JS中使用过了。而和现在 polyfills 的区别就是去年增加的 HTML5 polyfills。

相关信息：

* [Selectivizr][8]
* [CSS3Pie][9]
* [Modernizr wiki list of polyfills][10]

## **7.调试**##

主要是手机硬件方面的调试。
相关信息：

* [All of Chrome's command line switches][11]
* [Thomas Fuchs: Safari and iPhone Simulator CoreAnimation debugging info.][12]
* [HTML5Rocks: Improving the Performance of your HTML5 App][13]

[1]: https://dl.dropboxusercontent.com/u/39519/talks/cssperf/index.html
[2]: http://gent.ilcore.com/2011/03/how-not-to-trigger-layout-in-webkit.html
[3]: http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/
[4]: http://calendar.perfplanet.com/2009/the-new-game-show-will-it-reflow/
[5]: http://www.browserscope.org/?category=reflow
[6]: https://www.youtube.com/watch?v=a2_6bGNZ7bA
[7]: http://screwlewse.com/2010/08/different-css-techniques-and-their-performance/
[8]: http://selectivizr.com/
[9]: http://css3pie.com/
[10]: https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills
[11]: http://peter.sh/experiments/chromium-command-line-switches/ 
[12]: http://mir.aculo.us/2011/02/08/visualizing-webkits-hardware-acceleration/
[13]: http://www.html5rocks.com/en/tutorials/speed/html5/