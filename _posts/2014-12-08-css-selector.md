---
layout: post
category : css
title: 译.css选择器渲染优化
tags : [css, 翻译]
---
{% include JB/setup %}

原文：[css selector performance has changed][2]。这里来看看这篇文章的主要内容。

下面是我们之前就知道的：

*  选择器匹配元素是从右到左，应该避免比较艰难匹配的选择器。
*  后代选择器是比较缓慢的，尤其是最右的选择器匹配到大量元素的时候。

改变的现状：

Antti Koivisto，是`WebKit`代码的贡献者，最近花了一些时间去优化`css`的选择器匹配。在完成这项工作后，他说：

>  我的观点是：写代码的人不应该去担心选择器的优化，这应该是浏览器引擎的工作。

下面我们来看看他在四个方面做的特别优化：

1. Style Sharing（共享样式）
2. Rule Hashes（哈希规则）
3. Ancestor Filters（父代过滤器）
4. Fast Path（快速路径）

### **共享样式**
共享样式允许浏览器根据已经匹配到得元素，快速找出其他相同样式的元素。

ex: 

	<div>
	  <p>foo</p>
	  <p>bar</p>
	</div>

如果浏览器引擎已经计算出第一个`<p>`的样式（已经渲染），它就不用再去计算第二个`<p>`的样式，直接渲染就是了（当然这个前提是第一个`<p>`和第二个`<p>`的样式选择器一致，如`div p{}
`之类）。这是很简单的一小步，却优化
选择器的一大步。

### **哈希规则**
我们现在都知道匹配选择器是从右到左，所以最右边的选择器是很重要的。哈希规则基于最右的选择器来把选择器分组。举个例子，下面的选择器会分成三组。

	a {}
	div p {}
	div p.legal {}
	#sidebar a {}
	#sidebar p {}

	1: a, a {}, #sidebar a

	2: p, div p {}, #sidebar p {}

	3: p.legal , div p.legal {}

当浏览器使用哈希规则的时候，它不一定要去查找样式表中所个有的单的选择器，而是分组去匹配。
这对于页面上单个的`HTML`元素，同样很巧妙的节省了很多工作。

### **父代过滤器**
父代过滤器是有点复杂的。它们通过计算一个选择器可以匹配的可能性，去过滤，所以也称为可能性过滤器。所以，父代过滤器可以快速排除那些没有匹配的元素。它通过`class`,`id`,`tag`去匹配后代和子选择器。后代选择器是认为匹配比较慢的，因为渲染引擎要循环所有的父节点，尝试选择器是否匹配。这个时候`bloom`过滤器就闪亮登场了。

`bloom`过滤器是一个数据集合，可以让你判断某个选择器是否在集合中.bloom过滤器判断一个
css规则是否是在当前的元素集合中（The bloom filter tests whether a CSS rule is a member of the set of rules which match the element you are currently testing）。这其中很特别的一件事是，误报是有可能的，但漏报是不可能的。这意味着，如果bloom过滤器判断出一个选择器不符合当前的元素，浏览器就会停止查看当前的选择器，跳到下一个选择器。这里又节省了很多时间。如果`bloom`过滤器判断出当前的元素匹配，浏览器会继续通过常规的方法去进行100%的准确匹配。样式表文件越大，误报的几率就越大。所以要保持样式表文件的比较近小。

### **快速路径**
快速路径通过内在的循环，而不是递归，去取代常规的匹配逻辑（Fast path re-implements more general matching logic using a non-recursive, fully inlined loop. ）。 它通常用在匹配下面组合的选择器：

1. 后代，子代，子代选择器组合
2. `tag`，`id`，`class`和属性组合选择器

以上就是Antti对`WebKit`引擎的四个优化。(有点高深，不是很懂~囧)


[1]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Writing_efficient_CSS
[2]: http://calendar.perfplanet.com/2011/css-selector-performance-has-changed-for-the-better/