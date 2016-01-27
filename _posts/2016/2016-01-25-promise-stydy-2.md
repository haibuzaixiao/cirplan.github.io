---
layout: post
category : js
title: Promise初级教程2：详细介绍
tags : [web, js, Promise]
---
{% include JB/setup %}

上一篇简单介绍了 Promise 的外形，让大家了解 Promise 是什么样的，可以用在哪里，有个心理准备。不然突然跳出个恐龙出来，吓你们一跳，233。
其实这篇文章才算正式介绍 Promise，咳咳...

    本来想周末写的，不过遇到了广东百年难遇的下雪天气，一直窝在床上发抖，所以拖到现在（好吧，其实是懒）。

Promise 已经有很多人介绍了，这里安利一些比较值得看的文章，不妨先看看。

* [JavaScript Promises][5]（如果打不开，请翻墙）；
* 阮大大的 [Promise对象][4]；
* 网友翻译的 [JavaScript Promise迷你书][3]。

估计上面的看完，对 Promise 的大概都懂了，那接下来的内容，理论上可以不用看了，因为上面的已经说的很全很详细了～～不过，我还是按个人理解，整理一下。

## 1. Promise 定义

### 1.1 是什么

Promise 是抽象异步处理对象以及对其进行各种操作的组件 -- 来自 [JavaScript Promise迷你书][3]。

### 1.2 规范

ES6 的[Promise][0]是遵循 [Promises/A+][1] 规范的。Promises/A+ 一开始是社区规范，然后大部分的 Promise 类库遵循了这个规范，所以渐渐变成了一个标准。
所以我们可以通过 Promises/A+ 的规范去学习 Promise。

[Promises/A+中文版翻译][2]在这里，英文看不太懂可以看看翻译版本。

### 1.3 状态

每个 Promise 对象有一种状态：Pending（等待）、Resolved/Fulfilled（成功）和 Rejected（失败）中的一种，变化过程如下：

![promise_state](/images/2016/20160122promise/promise_states.png)

如上图，状态只能从 Pending 变为 Resolved 或者 Rejected 两者之中的一种，**过程不可以逆转**。

## 2. Promise 用法

这里主要介绍 Promise 的一些方法。

### 2.1 new Promise

通过 new，新建 Promise 对象，这是最常规的用法。看上一篇文章的例子：

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

结果很明显，是进入成功承诺的回调，这里没什么好说的。

### 2.2 Promise.resolve

静态方法 `Promise.resolve(value)` 是 `new Promise()` 的快捷方式。

    Promise.resolve(42); 

    // 等价于
    new Promise(function(resolve){
        resolve(42);
    });

#### 问题：参数 value 可以是什么？

1. 空。
2. 普通的对象。
3. 带有 then 方法的不标准 Promise 对象。
4. Promise 对象。

下面让我们一个个来验证，下面结果是在 chrome & firefox 里的调试结果。

#### 2.2.1 空

**如果不传参数**，则返回一个 value 为 undefined 的 Promise 对象。

    // value 为空，返回的Promise对象value为undefined
    Promise.resolve() 
    // chrome  --> Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: undefined}
    // firefox --> Promise { <state>: "fulfilled", <value>: undefined }

#### 2.2.2 普通的对象

**如果参数为普通对象**，返回状态为 Resolve 的 Promise 对象，其 value 为传人的参数。

    // value 为string
    Promise.resolve('42') 
    // chrome  --> Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: "42"}
    // firefox --> Promise { <state>: "fulfilled", <value>: "42" }

    // value 为 object
    Promise.resolve({a:1}) 
    // chrome  --> Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: Object}
    // firefox --> Promise { <state>: "fulfilled", <value>: Object }

#### 2.2.3 带有 then 方法的不标准 Promise 对象

**如果参数为带有then方法的不标准 Promise 对象**，返回新的 Promise 对象。我们先来看 jQuery 的 `$.ajax` 方法返回的 deferred 对象。
页面引入 jQuery-2.1.4，如下：

![jQuery_ajax](/images/2016/20160122promise/ajax.png)

通过上图，我们可以知道 deferred 对象是 thenable（包含了then方法的对象或函数）对象，但不是标准的 Promise 对象（怎样才算标准的Promise对象？）。

那看看通过 Promise.resolve 转换效果，先看请求成功的返回结果，可以看到 `[[PromiseValue]]` 就是 $.ajax 的 deffered 对象，如下：

![resolve_1](/images/2016/20160122promise/promise_resolve_1.png)

再看看请求错误的时候结果，如下：

![resolve_2](/images/2016/20160122promise/promise_resolve_2.png)

有没有看出什么？是不是成功或失败，返回 Promise 对象的状态都是 resolved ？那是不是都进入 resolved 的回调？想一下好像是的哦，让我们来验证下～

先看请求成功的时候，其实没什么悬念：

    var handler = Promise.reject($.ajax('/stock/chart/1414672685/position'));
    handler.then(function (val) {
        console.log('resolve', val);
    }, function (e) {
        console.log('rejected', e);
    });

![resolve_3](/images/2016/20160122promise/promise_resolve_3.png)

再看看请求错误的时候：

    var handler = Promise.reject($.ajax('/stock/chart/1414672685/position123'));
    handler.then(function (val) {
        console.log('resolve', val);
    }, function (e) {
        console.log('rejected', e);
    });

![resolve_4](/images/2016/20160122promise/promise_resolve_4.png)

纳尼，**为什么进入 rejected 的回调了**？不是说好 Promise 的状态，一旦从 Pending 到 Resolved 或 Rejected 就不能改变了吗？你们说这是为什么？

有人说，看上面两张图中 Promise 打印出来的状态是 `Pending`，秘密所在。咳咳，其实 `Promise.then` 方法会返回一个新的 Promise 对象，方便链式调用而已。

其实这个问题与 `Promise` 的解析流程有关，我们待会再看这个问题～

#### 2.2.4 Promise 对象

**如果参数为 Promise 对象**，直接返回该 Promise 对象：

    var promise1 = new Promise(function (resolve, rejected) {
        resolve('done')
    });
    Promise.resolve(promise1);

    var promise1 = new Promise(function (resolve, rejected) {
        rejected('fail')
    });
    Promise.resolve(promise1);  

![resolve_5](/images/2016/20160122promise/promise_resolve_5.png)

### 2.2 Promise.reject

`Promise.reject(error)` 是和 `Promise.resolve(value)` 类似的静态方法，是 `new Promise()` 方法的快捷方式。

    Promise.reject(new Error("出错了"))

    // 等价于
    new Promise(function(resolve,reject){
        reject(new Error("出错了"));
    });

正常来说，Promise.reject 的参数都应该是error对象，这个没什么好说的。

### 2.3 Promise.then

`Promise.then` 是 Promise 里比较重要的一个方法，在 [Promises/A+][1] 规范中，很大部分的篇幅就介绍这个方法。

then 方法，第一个参数是 Resolved 状态的回调函数，第二个参数（可选）是 Rejected 状态的回调函数，调用后返回新的 Promise 对象。就这么看，是不是完了？

#### 我们主要关注下面的两点：

1. 规范。
2. 返回新 Promise 对象的状态是怎么决定的。

#### 2.3.1 规范

    Promise.then(onFulfilled, onRejected)

`onFulfilled` & `onRejected`  的特点：
    
* 可选。
* 必须是函数，不然忽略。
* `onFulfilled` 必须在 Promise 的 Resolve 状态后调用，Promise 的 value 为其第一个参数，只能被调用一次。
* `onRejected` 必须在 Promise 的 Rejected 状态后调用，Promise 的 reason 为其第一个参数，只能被调用一次。
* 当成函数般调用（this为undefined）。

对于一个Promise , `then` 可以被调用多次：

* 当 Promise `fulfilled` 后，所有 onFulfilled 都必须按照其注册顺序执行。
* 当 Promise `rejected` 后，所有 OnRejected 都必须按照其注册顺序执行。

#### 2.3.2 返回新 Promise 

    promise2 = promise1.then(onFulfilled, onRejected);

* 如果 onFulfilled 或 onRejected 返回了值 x, 则执行 Promise 解析流程 `[[Resolve]](promise2, x)`。
* 如果 onFulfilled 或 onRejected 抛出了异常 e, 则 promise2 应当以 e 为 reason 被拒绝。
* 如果 onFulfilled 不是一个函数且 promise1 已经 fulfilled，则 promise2 必须以 promise1 的值 fulfilled。
* 如果 OnReject 不是一个函数且 promise1 已经 rejected, 则 promise2 必须以相同的 reason 被拒绝。

好吧，上面这些都是照搬规范的东西而已（我只是个搬运工，哭），基本上看一遍就知道怎么回事了。

### 2.4 Promise.catch

`Promise.catch` 只是 `Promise.then(undefined, onRejected)` 方法的一个别名。

这里盗个例子：

    function taskA() {
        console.log("Task A");
    }
    function taskB() {
        console.log("Task B");
    }
    function onRejected(error) {
        console.log("Catch Error: A or B", error);
    }
    function finalTask() {
        console.log("Final Task");
    }

    var promise = Promise.resolve();
    promise
        .then(taskA)
        .then(taskB)
        .catch(onRejected)
        .then(finalTask);

![promise-then-catch-flow](/images/2016/20160122promise/promise-then-catch-flow.png)

特点：

* 错误具有冒泡性，一直到捕获为止。
* IE8 下 `promise.catch` 有保留字问题，用 `promise["catch"]` 则没问题。
* 如果没有发生错误，`catch`会自动被忽略。

### 2.5 Promise.all

Promise.all 接收一个 promise 对象的数组作为参数，返回一个新的 promise 对象，特点如下：

* 当数组内所有 promise 对象的状态为 Resolve，其状态才为 Resolve。
* 当数组内有一个 promise 对象的状态为 Rejected，其状态就为 Rejected（这点promise迷你书中的说法是有误的）。

看测试代码：

    // 第一种情况
    var promise1 = Promise.resolve(1);
    var promise2 = new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('error');
        }, 1000);
    });
    var promise3 = Promise.all([promise1, promise2]);
    promise3.then(function (val) {
        console.log('resolve', val);
    }, function (e) {
        console.log('reject', e);
    });
    // resolve [1, "error"]

    // 第二种情况
    var promise1 = Promise.reject(1);
    var promise2 = new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('error');
        }, 1000);
    });
    var promise3 = Promise.all([promise1, promise2]);
    promise3.then(function (val) {
        console.log('resolve', val);
    }, function (e) {
        console.log('reject', e);
    });
    // reject 1

### 2.6 Promise.race

Promise.all 的相对方法，只要数组内有一个 promise 对象的状态改变，其状态就改变。

看测试代码：

    var promise1 = Promise.resolve(1);
    var promise2 = new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('error');
        }, 1000);
    });
    var promise3 = Promise.race([promise1, promise2]);
    promise3.then(function (val) {
        console.log('resolve', val);
    }, function (e) {
        console.log('reject', e);
    });
    // resolve 1

到这里，Promise 的方法已经介绍完了。然而上面还有个问题没解决，让我们来看下 Promise 的解析流程。

本来想接着写的，不过真的太长了，怕大家看的头晕。

所以，有兴趣的话，请看下一篇 [Promise 的解析流程][6]。 

[1]: https://promisesaplus.com/
[2]: http://segmentfault.com/a/1190000002452115
[3]: http://liubin.org/promises-book/
[4]: http://es6.ruanyifeng.com/#docs/promise
[5]: http://www.html5rocks.com/zh/tutorials/es6/promises/
[6]: /js/2016/01/26/promise-study-3/
