---
layout: post
category : js
title: Promise初级教程3：解析流程
tags : [web, js, Promise]
---
{% include JB/setup %}

接上篇，下面的序号是为了与[规范][1]中一致。

## 2.3. Promise 解析流程

这里主要是根据规范来看，要注意哈，搬运工又来了。

Promise 解析过程 是以一个 promise 和一个值 x 做为参数的抽象过程，可表示为 `[[Resolve]](promise, x)`, 过程如下：

##### 2.3.1 如果 promise 和 x 指向相同的值, 使用 TypeError 做为原因将 promise 拒绝，如下：

    var promise = Promise.resolve('done').then(function () {
        return promise;
    });

    promise.then(null, function (reason) {
        console.log('error', reason);
    });
    // error TypeError: Chaining cycle detected for promise #<Promise>

##### 2.3.2 如果 x 是一个 promise , 采用其状态:

* 2.3.2.1 如果 x 是 pending 状态，promise 必须保持 pending 走到 x fulfilled 或 rejected。
* 2.3.2.2 如果 x 是 fulfilled 状态，将 x 的值用于 fulfill promise。
* 2.3.2.3 如果 x 是 rejected 状态, 将 x 的原因用于 reject promise。

如下：

    var dummy = { dummy: "dummy" };
    
    // 2.3.2.1，x 是 pending 状态
    function xFactory() {
        return new Promise(function (resolve, reject) {
            setTimeout(function () {
                resolve(1000);
            }, 1000);
        });
    }

    var promise = Promise.resolve(dummy).then(function () {
        return xFactory();
    });

    promise.then(
        function (val) {
            console.log('resolve', val);
        },
        function (e) {
            console.log('reject', e);
        }
    );
    // resolve 1000

    // 2.3.2.2，x 是 fulfilled 状态
    var promise = Promise.resolve(dummy).then(function () {
        return Promise.resolve('done');
    });

    promise.then(
        function (val) {
            console.log('resolve', val);
        },
        function (e) {
            console.log('reject', e);
        }
    );
    // resolve done

    // 2.3.2.3，x 是 rejected 状态
    var promise = Promise.resolve(dummy).then(function () {
        return Promise.reject('error');
    });

    promise.then(
        function (val) {
            console.log('resolve', val);
        },
        function (e) {
            console.log('reject', e);
        }
    );
    // reject error

##### 2.3.3 如果 x 是一个对象或一个函数:

2.3.3.1 将 then 赋为 x.then
    
    // 1. 当 x 为没有原型的对象时
    var numberOfTimesThenWasRetrieved = 0;

    function xFactory() {
        return Object.create(null, {
            then: {
                get: function () {
                    ++numberOfTimesThenWasRetrieved;
                    return function thenMethodForX(onFulfilled) {
                        onFulfilled();
                    };
                }
            }
        });
    }

    var promise = Promise.resolve(dummy).then(function onBasePromiseFulfilled() {
        return xFactory();
    });

    console.log(promise);
    // Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}

    promise.then(function () {
        console.log(numberOfTimesThenWasRetrieved);
    });
    // 1

    // 2. 当 x 为有基本原型的对象，修改如下
    function xFactory() {
        return Object.create(Object.prototype, {
            then: {
                get: function () {
                    ++numberOfTimesThenWasRetrieved;
                    return function thenMethodForX(onFulfilled) {
                        onFulfilled();
                    };
                }
            }
        });
    }
    // 1

    // 3. 当 x 为函数时
    function xFactory() {
        function x() { }

        Object.defineProperty(x, "then", {
            get: function () {
                ++numberOfTimesThenWasRetrieved;
                return function thenMethodForX(onFulfilled) {
                    onFulfilled();
                };
            }
        });

        return x;
    }
    // 1

2.3.3.2 如果在取 x.then 值时抛出了异常，则以这个异常做为原因将 promise 拒绝

    // 取 x.then 的时候，出现错误
    function xFactory() {
        return Object.create(Object.prototype, {
            then: {
                get: function () {
                    throw Error('error');
                }
            }
        });
    }

![promise_catch_1](/images/2016/20160122promise/promise_catch_1.png)
    
2.3.3.3 如果 then 是一个函数， 以 x 为 this 调用 then 函数，且第一个参数是 resolvePromise，第二个参数是 rejectPromise，且：

* 2.3.3.3.1 当 resolvePromise 被以 y 为参数调用, 执行 `[[Resolve]](promise, y)`。
* 2.3.3.3.2 当 rejectPromise 被以 r 为参数调用, 则以 r 为原因将 promise 拒绝。
* 2.3.3.3.3 如果 resolvePromise 和 rejectPromise 都被调用了，或者被调用了多次，则只第一次有效，后面的忽略。
* 2.3.3.3.4 如果在调用 then 时抛出了异常，则。
    * 2.3.3.3.4.1 如果 resolvePromise 或 rejectPromise 已经被调用了，则忽略它。
    * 2.3.3.3.4.2 否则, 以 e 为 reason 将 promise 拒绝。

这个有点绕，看测试代码也挺复杂。这里就不贴代码了（测试代码900多行），有兴趣直接看 [这里][2]。

所以，回到之前的问题：

![resolve_4](/images/2016/20160122promise/promise_resolve_4.png)

这个看 2.3.3.4，因为请求错误的接口，在调用 then 时抛出了异常，所以会进入到 Rejected 状态。

哎，感觉这个流程还是有点问题，再琢磨琢磨。

[1]: https://promisesaplus.com/
[2]: https://github.com/promises-aplus/promises-tests/blob/master/lib/tests/2.3.3.js





