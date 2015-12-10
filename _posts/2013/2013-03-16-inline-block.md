---
layout: post
category : CSS
title: IE6/IE7下：inline-block解决方案
tags : [inline-block, html]
---
{% include JB/setup %}

IE6/IE7下对`display:inline-block`的支持性不好。

* `inline`元素的display属性设置为`inline-block`时，所有的浏览器都支持；
* `block`元素的display属性设置为`inline-block`时，IE6/IE7浏览器是不支持的；


		对象呈递为内联对象，但是对象的内容作为块对象呈递。旁边的内联对象会被呈递在同一行，允许空格。（准确地说，应用此特性的元素现为内联对象，周围元素保持在同一行，但可以设置宽度和高度等块元素的属性）

		IE中对内联元素使用display:inline-block，IE是不识别的，但使用display:inline-block在IE下会触发layout，从而使内联元素拥有了display:inline-block属性的表征。从上面的这个分析，也不难理解为什么IE下，对块元素设置display:inline-block属性无法实现inline-block的效果。这时块元素仅仅是被display:inline-block触发了layout，而它本身就是行布局，所以触发后，块元素依然还是行布局，而不会如Opera中块元素呈递为内联对象。


IE6下块元素如何实现`display:inline-block`的效果？

有两种方法：

* 1 先使用display:inline-block属性触发块元素，然后再定义display:inline，让块元素呈递为内联对象（两个display 要先后放在两个CSS声明中才有效果，这是IE的一个经典bug，如果先定义了display:inline-block，然后再将display设回 inline或block，layout不会消失）。代码如下（...为省略的其他属性内容）：

		div {display:inline-block;...}
		div {display:inline;}

* 2 直接让块元素设置为内联对象呈递（设置属性display:inline），然后触发块元素的layout（如：zoom:1 或float属性等）。代码如下：

		div { display:inline-block; _zoom:1;_display:inline;} /*推荐*/
		div { display:inline-block; _zoom:1;*display:inline;} /*推荐:IE6/7*/
