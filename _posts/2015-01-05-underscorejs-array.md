---
layout: post
category : api
title: underscore.js Api -- array
tags : [js, underscore, api]
---
{% include JB/setup %}

今天看[underscore.js][1]的collections部分。

Note: 所有数组函数对于 arguments 对象一样有用，不只是针对稀疏数组。

导航：

* [first](#first)
* [initial](#initial)
* [last](#last)
* [compact](#compact)
* [flatten](#flatten)
* [without](#without)
* [union](#union)
* [intersection](#intersection)
* [difference](#difference)
* [uniq](#uniq)
* [zip](#zip)
* [object](#object)
* [indexOf](#indexOf)
* [lastIndexOf](#lastIndexOf)
* [sortedIndex](#sortedIndex)
* [range](#range)

### first
` _.first(array, [n]) Alias: head, take  `

* 没有传参数n，返回数组的第一个元素
* 传参数n，返回数组的前n个元素

		_.first([5, 4, 3, 2, 1]);
		=> 5

### initial
` _.initial(array, [n]) `

* 没有参数n，返回数组中除最后一个元素之外的其他元素
* 传入参数n，返回数组中除最后n个元素之外的其他元素

		_.initial([5, 4, 3, 2, 1]);
		=> [5, 4, 3, 2]

### last
` _.last(array, [n]) `

* 没有参数n，返回数组的最后一个元素
* 有参数n，返回数组最后的n个元素

		_.last([5, 4, 3, 2, 1]);
		=> 1

### rest
` _.rest(array, [index]) Alias: tail, drop `

* 没有参数index，返回数组中除第一个元素的其他元素
* 有参数index，返回数组中除indx位置元素的其他元素

		_.rest([5, 4, 3, 2, 1]);
		=> [4, 3, 2, 1]

### compact 
` _.compact(array) `

* 返回一个数组的副本，除去非空元素
* 被判定为空的为：false, null, 0, "", undefined and NaN 

		_.compact([0, 1, false, 2, '', 3]);
		=> [1, 2, 3]

### flatten 
` _.flatten(array, [shallow]) `

* 没有shallow，把嵌套多层的元素转换成只有一层的。
* shallow 为true时，只是去掉元素的一层嵌套

		_.flatten([1, [2], [3, [[4]]]]);
		=> [1, 2, 3, 4];

		_.flatten([1, [2], [3, [[4]]]], true);
		=> [1, 2, 3, [[4]]];

###without
` _.without(array, *values) `

* 返回排除所有values值得数组副本

		_.without([1, 2, 1, 0, 3, 1, 4], 0, 1);
		=> [2, 3, 4]

### union
` _.union(*arrays) `

* 返回结合后的数组：每个元素都是独一无二的。可以传入一个或多个数组。

		_.union([1, 2, 3], [101, 2, 1, 10], [2, 1]);
		=> [1, 2, 3, 101, 10]

### intersection
` _.intersection(*arrays) `

* 返回一个或多个数组中元素的交集，即返回一个的元素在每一个数组里

		_.intersection([1, 2, 3], [101, 2, 1, 10], [2, 1]);
		=> [1, 2]

### difference 
` _.difference(array, *others) `

* 和without类似，但返回的是array中吧不在others数组中的元素

		_.difference([1, 2, 3, 4, 5], [5, 2, 10]);
		=> [1, 3, 4]

### uniq
` _.uniq(array, [isSorted], [iteratee]) Alias: unique  `

* 数组去重，用 === 进行测试。
* 如果数组已排序，传入isSorted 为true，这样这个方法的速度会快很多。
* 也可传入iteratee去比较元素是否相同

		_.uniq([1, 2, 1, 3, 1, 4]);
		=> [1, 2, 3, 4]

### zip
` _.zip(*arrays) `

* 合并每个数组的每个元素，并保留对应位置。
* 在合并分开保存的数据时很常用。
* 如果处理矩阵相似数组，_.zip.apply会达到相同的效果。

		_.zip(['moe', 'larry', 'curly'], [30, 40, 50], [true, false, false]);
		=> [["moe", 30, true], ["larry", 40, false], ["curly", 50, false]]

		_.zip.apply(_, arrayOfRowsOfData);
		=> arrayOfColumnsOfData

### object 
` _.object(list, [values]) `

* 把数组转变成对象。
* 数组的元素可以为[key, value]形式，也可以一个数组的元素全为key，一个数组的元素全为value。
* 如果存在重复key，后一个会被应用。

		_.object(['moe', 'larry', 'curly'], [30, 40, 50]);
		=> {moe: 30, larry: 40, curly: 50}

		_.object([['moe', 30], ['larry', 40], ['curly', 50]]);
		=> {moe: 30, larry: 40, curly: 50}

### indexOf
` _.indexOf(array, value, [isSorted]) `

* 返回元素在数组中的索引，没有则返回-1。
* 如果数组已排序，传入isSorted为true，这样会采用更快速的二分排序法查找元素。
* 可以传入一个数字，以便在指定索引之后寻找对应值。

		_.indexOf([1, 2, 3], 2);
		=> 1

### lastIndexOf
` _.lastIndexOf(array, value, [fromIndex]) `

* 返回元素在数组中最后出现的位置，没有则返回-1。
* 传入fromIndex则在指定位置开始查找。

		_.lastIndexOf([1, 2, 3, 1, 2, 3], 2);
		=> 4

### sortedIndex
` _.sortedIndex(list, value, [iteratee], [context]) `

* 为了保持list的顺序，采用二分搜索去查找value应该插入到list中对应的值。
* 如果传入iteratee，则用来计算每个元素的值，包括value。

		_.sortedIndex([10, 20, 30, 40, 50], 35);
		=> 3

		var stooges = [{name: 'moe', age: 40}, {name: 'curly', age: 60}];
		_.sortedIndex(stooges, {name: 'larry', age: 50}, 'age');
		=> 1

### range
` _.range([start], stop, [step]) `

* 一个简单创建整数数组的函数，each和map循环的简便版本。
* start默认为0，step默认为1。
* 返回一个数组，包含从start到stop内（不包括stop）以step递增（递减）的函数。

		_.range(10);
		=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
		_.range(1, 11);
		=> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
		_.range(0, 30, 5);
		=> [0, 5, 10, 15, 20, 25]
		_.range(0, -10, -1);
		=> [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
		_.range(0);
		=> []




[1]: http://underscorejs.org/