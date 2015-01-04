---
layout: post
category : api
title: underscore.js Api -- collections
tags : [js, underscore, api]
---
{% include JB/setup %}

> 转眼到了2015年，这一年还是要努力啊，赚钱赚钱赚钱。

今天看[underscore.js][1]的collections部分。

导航：

* [each](#each)
* [map](#map)
* [reduce](#reduce)
* [reduceRight](#reduceRight)
* [find](#find)
* [filter](#filter)
* [where](#where)
* [findWhere](#findWhere)
* [reject](#reject)
* [every](#every)
* [some](#some)
* [contains](#contains)
* [invoke](#invoke)
* [pluck](#pluck)
* [max](#max)
* [min](#min)
* [sortBy](#sortBy)
* [groupBy](#groupBy)
* [indexBy](#indexBy)
* [countBy](#countBy)
* [shuffle](#shuffle)
* [sample](#sample)
* [toArray](#toArray)
* [size](#size)
* [partition](#partition)

## Collection Functions (Arrays or Objects)  集合函数，参数是数组或对象

Tip: Collection函数的参数可以是array， objects, 和array-like objects(类似数组对象)，
就如arguments(函数参数)，NodeList(Dom节点列表)和类似的对象。但由于其工作方式是duck-typing(鸭子模式？)，
所以避免传入的对象有length属性。

#### each
`_.each(list, iteratee, [context])，Alias(别名): forEach`

* 对于list里的每一个元素进行iteratee(迭代)函数的操作。
* 当list是数组，iteratee的参数是(element, index, list)。
* 当list是对象，iteratee的参数是(value, key, list)。
* each循环是不能中断的，如果要中断，可以用_.find代替
* 返回原list

		_.each([1, 2, 3], function(element, index, list) { });
		_.each({one: 1, two: 2, three: 3}, function(value, key, list) { });


### map
`_.map(list, iteratee, [context]) Alias: collect`

* 和each很相似，参数都一样。
* 返回的是处理后的数组

		_.map([1, 2, 3], function(num){ return num * 3; });
		=> [3, 6, 9]
		_.map({one: 1, two: 2, three: 3}, function(num, key){ return num * 3; });
		=> [3, 6, 9]

### reduce
`_.reduce(list, iteratee, [memo], [context]) Aliases: inject, foldl`

* reduce把list里的所有值迭代成一个值。
* memo是迭代的初始值，参与每次迭代。如果没有定义memo的初值，memo则为list的第一个值。迭代从list的第二个元素开始。
* iteratee的参数(memo, value, index, list)
* 返回迭代的最终值。

		var sum = _.reduce([1, 2, 3], function(memo, num){ return memo + num; }, 0);
		=> 6

### reduceRight
`_.reduceRight(list, iteratee, memo, [context]) Alias: foldr`

* reduce的右结合版本。
* 如果javascript原生存在(v1.8)，则调用原生的reduceRight。

		var list = [[0, 1], [2, 3], [4, 5]];
		var flat = _.reduceRight(list, function(a, b) { return a.concat(b); }, []);
		=> [4, 5, 2, 3, 0, 1]

### find 
` _.find(list, predicate, [context]) Alias: detect `

* predicate的参数是`(value, key, list)`。
* 返回list中，第一个通过predicate验收的值。如果所有值都不通过验证，则返回undefined。


		var even = _.find([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
		=> 2

### filter
` _.filter(list, iterator, [context]) 别名: select `

* find的全局版本，返回list中所有通过predicate验证的的数组。

		var evens = _.filter([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
		=> [2, 4, 6]

### where
` _.where(list, properties) `

* 对list中的每一个元素进行检查，返回数组。
* 返回数组里的每一项都包含properties的所有属性。

		_.where(listOfPlays, {author: "Shakespeare", year: 1611});
		=> [{title: "Cymbeline", author: "Shakespeare", year: 1611},
		    {title: "The Tempest", author: "Shakespeare", year: 1611}]

### findWhere
` _.findWhere(list, properties) `

* where的单一版本，返回list中第一个包含properties所有属性的元素。
* 如果没有，返回undefined。

		_.findWhere(publicServicePulitzers, {newsroom: "The New York Times"});
		=> {year: 1918, newsroom: "The New York Times",
		  reason: "For its public service in publishing in full so many official reports,
		  documents and speeches by European statesmen relating to the progress and
		  conduct of the war."}

### reject
`_.reject(list, predicate, [context]) ` 

* filter的相反函数，返回list中不符合predicate函数的值。

		var odds = _.reject([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
		=> [1, 3, 5]

### every
` _.every(list, [predicate], [context]) Alias: all `

* 如果list中的所有值都通过predicate验证，返回true，否则返回false。

		_.every([true, 1, null, 'yes'], _.identity);
		=> false

### some 
` _.some(list, [predicate], [context]) Alias: any `

* 如果list中的任意一个元素通过predicate验证，停止并返回true，否则返回false。

		_.some([null, 0, 'yes', false]);
		=> true

### contains
` _.contains(list, value) Alias: include  `

* 如果value在list中，返回true。否则返回false。
* 如果list是数组，调用的是indexOf方法。

		_.contains([1, 2, 3], 3);
		=> true

### invoke
` _.invoke(list, methodName, *arguments) `

* 对于list中的每一个元素，调用方法名为methodName的函数。
* 传入附加参数，会转给要调用的函数。

		_.invoke([[5, 1, 7], [3, 2, 1]], 'sort');
		=> [[1, 5, 7], [1, 2, 3]]

### pluck 
` _.pluck(list, propertyName) `

* map的简单版本。
* 返回list中指定属性的值的数组。

		var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
		_.pluck(stooges, 'name');
		=> ["moe", "larry", "curly"]

###max 
` _.max(list, [iteratee], [context]) `

* 返回list中最大值的value。
* 如果传入iteratee函数，则以iteratee函数为标准比较。

		var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
		_.max(stooges, function(stooge){ return stooge.age; });
		=> {name: 'curly', age: 60};

### min
` _.min(list, [iteratee], [context]) `

* max的想反函数。返回list中的最小项。

		var numbers = [10, 5, 100, 2, 1000];
		_.min(numbers);
		=> 2

###sortBy
` _.sortBy(list, iteratee, [context]) `

* 返回的是经过排序的list。
* 排序的标准是通过比较iteratee函数返回值的大小，或者是属性的名称。如：length。

		_.sortBy([1, 2, 3, 4, 5, 6], function(num){ return Math.sin(num); });
		=> [5, 4, 6, 3, 1, 2]

### groupBy
` _.groupBy(list, iteratee, [context]) `

* 把collection分成一个个的集合。
* 分组的标准是iteratee函数每个返回值；如果iteratee是字符串，则通过元素的属性值分组。

		_.groupBy([1.3, 2.1, 2.4], function(num){ return Math.floor(num); });
		=> {1: [1.3], 2: [2.1, 2.4]}

		_.groupBy(['one', 'two', 'three'], 'length');
		=> {3: ["one", "two"], 5: ["three"]}

### indexBy
` _.indexBy(list, iteratee, [context]) `

* 返回的是类似于groupBy的对象，带有索引。
* 对于list的任何的元素, iteratee函数返回一个独一无二key。

		var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
		_.indexBy(stooges, 'age');
		=> {
		  "40": {name: 'moe', age: 40},
		  "50": {name: 'larry', age: 50},
		  "60": {name: 'curly', age: 60}
		}

###countBy
` _.countBy(list, iteratee, [context]) `

* 把list分组，但返回的是每一组的数目。类似于groupBy。

	_.countBy([1, 2, 3, 4, 5], function(num) {
	  return num % 2 == 0 ? 'even': 'odd';
	});
	=> {odd: 3, even: 2}

### shuffle
` _.shuffle(list) `

* 返回随机乱序数组

	_.shuffle([1, 2, 3, 4, 5, 6]);
	=> [4, 1, 6, 3, 5, 2]

### sample
` _.sample(list, [n]) `

* 随机返回list中的n个元素，如果n没有定义，则返回一个元素。

		_.sample([1, 2, 3, 4, 5, 6]);
		=> 4

		_.sample([1, 2, 3, 4, 5, 6], 3);
		=> [1, 6, 2]

### toArray
` _.toArray(list) `

* 对于任何一个可以被迭代的list，返回一个数组。
* 对arguments对象的转换十分有用。

		(function(){ return _.toArray(arguments).slice(1); })(1, 2, 3, 4);
		=> [2, 3, 4]

### size
` _.size(list) `

* 返回list中元素的个数。

		_.size({one: 1, two: 2, three: 3});
		=> 3

### partition 
` _.partition(array, predicate) `

* 把array分成两组，一组满足predicate的验证，一组不满足predicate的验证。

		_.partition([0, 1, 2, 3, 4, 5], isOdd);
		=> [[1, 3, 5], [0, 2, 4]]


参考：

* [http://learningcn.com/underscore/][2]

[1]: http://underscorejs.org/
[2]: http://learningcn.com/underscore/



