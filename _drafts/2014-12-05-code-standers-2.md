---
layout: post
category : 编码规范
title: 前端编码规范（2）-- css
tags : [css, html]
---
{% include JB/setup %}

接着之前，再来看看css的编码规范。

## **编码总原则**

* 从外部文件加载`CSS`，尽可能减少文件数。加载标签必须放在文件的 `HEAD` 部分。
* 用 `LINK` 标签加载，永远不要用`@import`。
* 不要在文件中用内联式引入的样式，不管它是定义在样式标签里还是直接定义在元素上。这样会很难追踪样式规则。
* 使用 [normalize.css][1] 让渲染效果在不同浏览器中更一致。
* 使用类似 [YUI fonts.css][2] 的字体规格化文件。
* 理解 [层叠和选择器的明确度][3] [(翻译)][5] ，这样你就可以写出非常简洁且高效的代码。
* 编写性能优化的选择器。尽可能避免使用开销大的`CSS`选择器。例如，避免 `*` 通配符选择器，也不要叠加限定条件到 `ID` 选择器（例如 `div#myid`）或 `class` 选择器（例如 `table.results`）上。这对于速度至上并包含了成千上万个`DOM`元素的web应用来说尤为重要。更多相关内容请参阅 MDN 上的这篇 [《编写高效CSS》][4]。

## **CSS盒子模型**

![css-box-model](6)

## **CSS 格式**




[1]: http://necolas.github.io/normalize.css/
[2]: http://yui.github.io/yui2/
[3]: http://www.stuffandnonsense.co.uk/archives/css_specificity_wars.html
[4]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Writing_efficient_CSS
[5]: /2014/12/05/css-specificity-wars.html
[6]: /images/box_model.png
