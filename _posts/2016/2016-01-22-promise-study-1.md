---
layout: post
category : js
title: Promise初级教程1：简单使用
tags : [web, js, promise]
---
{% include JB/setup %}

就快过年了，项目组是准备在回去之前，发完2.4版本。然后现在已经进入测试后期了。

趁着现在有点时间，把之前一直想写的promise写一下。

	刚说完就来一个Bug，先去改bug。

过了N个半小时后，又回来了。让我们直接进入主题。

## Promise 是什么

**承诺**。就像有人问：你是男的吗？你可以做出俩种回答：是或者不是。

回答是，那就是肯定的承诺。

回答不是，那就是拒绝的承诺。

只能有俩种吗?是的，Promise里面只能有两种承诺，肯定或否定。

那我什么时候需要这样的承诺？感觉用不上啊？**在异步调用的时候**。

## Promise怎么用？

一般学习一个新东西，最直接的问题是怎么用。不然有可能造成看了一堆理论，还是云里雾里。引用一句话就是：道理我都懂，然后呢？

### 新建Promise对象

Promise作为构造函数，新建如下：

	var handler = new Promise(function (resolve, reject) {
        ... // 逻辑代码
	});

嗯，新建Promise对象很简单嘛，就传入一个函数作为参数，然后逻辑代码在函数里面。这个函数再带着 resolve 和 reject 俩个函数作为参数。
什么？resolve & reject是函数？没错，他们是函数。那他们有什么作用？

resolve 表示肯定承诺。

reject 表示否定承诺。

### then?

定义了Promise对象，然后呢？然后就要处理肯定或者否定的承诺啊。

	var handler = new Promise(function (resolve, reject) {
        ... // 逻辑代码
	}); 

	handler.then(function (val) {
        ... // 肯定承诺的回调
	}, function (e) {
        ... // 否定承诺的回调
	});

这里可以看到，`then(resolve, reject)`方法接收两个参数，分别是肯定承诺的回调和否定承诺的回调。

到这里，基本的Promise使用就介绍结束了。什么？还是不懂？我们看看例子。

### 例子

我们来看看这个例子，新建了Promise对象，500ms后进行了肯定的承诺。因为新版Chrome和Firefox是自带Promise，所以我们可以直接在控制台里测试。

肯定承诺例子：

	// ex1
	var startTime = Date.now();
	console.log('start', startTime);
	
	var handler = new Promise(function (resolve, reject) {
        setTimeout(function () {
        	resolve('done', Date.now() - startTime);
        }, 500);
	}); 

	handler.then(function (val) {
        console.log('then resolve', val, Date.now() - startTime);
	}, function (e) {
        console.log('then reject', e, Date.now() - startTime);
	});

![ex1](/images/2016/20160122promise/ex1.png)

嗯，在这个例子了，我们可以明显感受到 resolve 函数的用法。那么 reject 呢？其实是差不多的。

否定承诺例子：

    // ex2
	var startTime = Date.now();
    console.log('start', startTime);
    
    var handler = new Promise(function (resolve, reject) {
        setTimeout(function () {
            reject('done', Date.now() - startTime);
        }, 500);
    }); 

    handler.then(function (val) {
        console.log('then resolve', val, Date.now() - startTime);
    }, function (e) {
        console.log('then reject', e, Date.now() - startTime);
    });

![ex2](/images/2016/20160122promise/ex2.png)

在上面这个例子，给出了拒绝的承诺，然后就会执行 then 方法的 reject 回调函数。是不是很简单。

ok，到目前为止，你可以大声的说，你会用promise啦～不过这个只是简单的介绍，让大家初步了解而已。

下一篇，[介绍Promise的详细使用][1]。

[1]: /js/2016/01/25/promise-stydy-2/
