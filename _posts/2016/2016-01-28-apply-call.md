---
layout: post
category : js
title: JS中apply、call和bind的定义和使用
tags : [js, apply, call]
---
{% include JB/setup %}

本来没打算想写这个的，因为一直觉得 apply 和 call 挺简单的，不就是改变运行上下文嘛。然后 apply 和 call 只是参数接受的方式不同而已。

但在用的时候，总是会有点问题，囧。应该是没理解透吧，所以再琢磨琢磨，在这里记录下。

## 1. call [规范][1]

### 1.1 定义

    The call() method calls a function with a given this value and arguments provided individually.

call 方法通过给定的 this value 和一系列的参数，来调用函数。

### 1.2 Syntax 语法

`fun.call(thisArg[, arg1[, arg2[, ...]]])`

#### 1.2.1 Parameters 参数

**thisArg**

    The value of this provided for the call to fun. Note that this may not be the actual value seen by the method: if the method is a function in non-strict mode code, null and undefined will be replaced with the global object and primitive values will be converted to objects.

函数运行时的 this 值。要注意的是，有可能你看到的值并不是函数实际执行的值：如果函数是在`非严格模式`下，指定为 null 或 undefined 的 this 值会被替换成全局对象，
原始值（数字，字符串，布尔值）会被转为对象。看测试代码（可以直接粘到控制台执行）：

    // 1. 非严格模式
    // 1.1 null
    function log () {
        console.log('line1', this);
        console.log('line2', window === this);
    }

    log.call(null);
    // line1 Window {top: Window, location: Location, document: document, window: Window, external: Object…}
    // line2 true

    // 1.2 undefined
    log.call(undefined);
    // line1 Window {top: Window, location: Location, document: document, window: Window, external: Object…}
    // line2 true

    // 1.3 number
    function log () {
        console.log('line1', this);
        console.log('line2', this instanceof Number);
    }

    log.call(1);
    // line1 Number {[[PrimitiveValue]]: 1}
    // line2 true

    // 1.4 string
    function log () {
        console.log('line1', this);
        console.log('line2', this instanceof String);
    }

    log.call('str');
    // line1 String {0: "s", 1: "t", 2: "r", length: 3, [[PrimitiveValue]]: "str"}
    // line2 true

    1.5 boolbean
    function log () {
        console.log('line1', this);
        console.log('line2', this instanceof Boolean);
    }

    log.call(true);
    // line1 Boolean {[[PrimitiveValue]]: true}
    // line2 true

    ///////////////////////////////////////////////////////////////////////////////

    // 2. 严格模式，this 为指定值
    (function () {
        "use strict";

        function say () {
            console.log(this);
        }

        say.call(null);

    })();
    // null

**arg1, arg2, ...**

    Arguments for the object.

方法的参数。

### 1.3 Description 描述

    A different this object can be assigned when calling an existing function. this refers to the current object, the calling object. With call, you can write a method once and then inherit it in another object, without having to rewrite the method for the new object.

方法可以被不同的对象调用。this 会被指定为当前调用对象。通过 call ，你只需要写一次方法，就可以被其他对象继承，而不需要为新对象重写方法。

### 1.4 Examples 例子

**1.4.1 Using call to chain constructors for an object.**

让对象链接构造函数（好别扭，不知道是不是这样翻译）？

    function Product(name, price) {
      this.name = name;
      this.price = price;

      if (price < 0) {
        throw RangeError('Cannot create product ' +
                          this.name + ' with a negative price');
      }
    }

    function Food(name, price) {
      Product.call(this, name, price);
      this.category = 'food';
    }

    function Toy(name, price) {
      Product.call(this, name, price);
      this.category = 'toy';
    }

    var cheese = new Food('feta', 5);
    var fun = new Toy('robot', 40);
    // Food {name: "feta", price: 5, category: "food"}
    // Toy {name: "robot", price: 40, category: "toy"}
    
作用：复制构造函数的属性，方法。

那么能不能获得构造函数**原型**上的属性和方法呢？答案是肯定不行的，因为没有原型链。这一点要谨记。

    function Product(name, price) {
      this.name = name;
      this.price = price;

      if (price < 0) {
        throw RangeError('Cannot create product ' +
                          this.name + ' with a negative price');
      }
    }

    Product.prototype.sayName = function () {
        console.log(this.name);
    }

    function Food(name, price) {
      Product.call(this, name, price);
      this.category = 'food';
    }

    var cheese = new Food('feta', 5);
    cheese.sayName();

![call_error](/images/2016/20160128call/call_error.png)   

**1.4.2 Using call to invoke an anonymous function.**

通过 call 来调用匿名函数。

    var animals = [
      { species: 'Lion', name: 'King' },
      { species: 'Whale', name: 'Fail' }
    ];

    for (var i = 0; i < animals.length; i++) {
      (function(i) {
        this.print = function() {
          console.log('#' + i + ' ' + this.species
                      + ': ' + this.name);
        }
        this.print();
      }).call(animals[i], i);
    }
    // #0 Lion: King
    // #1 Whale: Fail

**1.4.3 Using call to invoke a function and specifying the context for 'this'.**

通过 call 来调用一个方法，并指定上下文 this。

    function greet() {
      var reply = [this.person, 'Is An Awesome', this.role].join(' ');
      console.log(reply);
    }

    var i = {
      person: 'Douglas Crockford', role: 'Javascript Developer'
    };

    greet.call(i); // Douglas Crockford Is An Awesome Javascript Developer


## 2. apply [规范][2]

apply 的作用和 call 一样，只是在调用的时候，传参数的方式不同。call 中的参数要一个一个按传进来，而 apply 中的参数是数组的形式。

### 2.1 Syntax 语法

    fun.apply(thisArg, [argsArray])

### 2.2 Examples 例子

因为和 call 太相似，这里主要看看例子，感受下不同之处。

**2.2.1 Using apply to chain constructors.**

使用 apply 链构造函数。

    Function.prototype.construct = function (aArgs) {
      var oNew = Object.create(this.prototype);
      this.apply(oNew, aArgs);
      return oNew;
    };

    function MyConstructor() {
      for (var nProp = 0; nProp < arguments.length; nProp++) {
        this['property' + nProp] = arguments[nProp];
      }
    }

    var myArray = [4, 'Hello world!', false];
    var myInstance = MyConstructor.construct(myArray);

    console.log(myInstance.property1);                // logs 'Hello world!'
    console.log(myInstance instanceof MyConstructor); // logs 'true'
    console.log(myInstance.constructor);     

**2.2.2 Using apply and built-in functions*.**

通过 apply 使用内置函数。这个主要是 Math.max / Math.min 之类的内置函数。

    // min/max number in an array
    var numbers = [5, 6, 2, 3, 7];

    // using Math.min/Math.max apply
    var max = Math.max.apply(null, numbers); 
    // This about equal to Math.max(numbers[0], ...)
    // or Math.max(5, 6, ...)

    var min = Math.min.apply(null, numbers);

    // vs. simple loop based algorithm 对比循环方式
    max = -Infinity, min = +Infinity;

    for (var i = 0; i < numbers.length; i++) {
      if (numbers[i] > max) {
        max = numbers[i];
      }
      if (numbers[i] < min) {
        min = numbers[i];
      }
    }

**2.2.3 Using apply in "monkey-patching"**

用于猴子补丁（what）？用在哪里？解析说这是一种hack。

    var originalfoo = someobject.foo;
    someobject.foo = function() {
      // Do stuff before calling function
      console.log(arguments);
      // Call the function as it would have been called normally:
      originalfoo.apply(this, arguments);
      // Run stuff after, here.
    }

## 3. bind [规范][3]

bind 是 ES5 中新增的一个方法，也是改变运行的上下文。那么与 call 和 apply 有什么区别呢？我们来看看～

### 3.1 定义

    The bind() method creates a new function that, when called, has its this keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called.

bind 方法在调用的时候，会新建一个函数，参数为 value 和一些列提供的参数，这个函数在调用的时候，this 会设置为参数中的 value（一直翻的不通顺）。

### 3.2 Syntax 语法

`fun.bind(thisArg[, arg1[, arg2[, ...]]])`

#### 3.2.1 Parameters 参数

**thisArg**

    The value to be passed as the this parameter to the target function when the bound function is called. The value is ignored if the bound function is constructed using the new operator.

当绑定函数在调用的时候，这个值将作为 this 参数传递给目标函数。如果绑定函数使用了 new 操作符，这个值会被忽略。

**arg1, arg2, ...**

    Arguments to prepend to arguments provided to the bound function when invoking the target function.

在目标函数调用的时候，传递给绑定函数的参数。

### 3.3 Description 描述

    The bind() function creates a new function (a bound function) with the same function body (internal call property in ECMAScript 5 terms) as the function it is being called on (the bound function’s target function) with the this value bound to the first argument of bind(), which cannot be overridden. bind() also accepts leading default arguments to provide to the target function when the bound function is called. A bound function may also be constructed using the new operator: doing so acts as though the target function had instead been constructed. The provided this value is ignored, while prepended arguments are provided to the emulated function.

bind 方法在调用的时候，会创建一个新的函数（绑定函数），函数体和调用的函数一样，其中 this 值为 bind 方法的第一个参数，不能被重写。
绑定函数在调用的时候，也会接受目标函数的默认参数。绑定函数也可以通过 new 操作符构建：就好像目标函数被构建函数替换了（？）。提供的 this 值会被忽略（看不懂）。

### 3.4 Examples 例子

#### 3.4.1 Creating a bound function 

创建绑定函数。最基本的绑定函数方式。

    this.x = 9; 
    var module = {
      x: 81,
      getX: function() { return this.x; }
    };

    module.getX(); // 81

    var retrieveX = module.getX;
    retrieveX(); // 9, 因为 this 指向全局对象

    // 创建一个新的函数，其中 this 指向 module
    var boundGetX = retrieveX.bind(module);
    boundGetX(); // 81

#### 3.4.2 Partial Functions 

部分函数。可以建立有默认参数的函数。

    function list() {
      return Array.prototype.slice.call(arguments);
    }

    var list1 = list(1, 2, 3); // [1, 2, 3]

    // 创建一个有预置参数的函数
    var leadingThirtysevenList = list.bind(undefined, 37);

    var list2 = leadingThirtysevenList(); // [37]
    var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]

#### 3.4.3 With setTimeout 

在调用 setTimeout 时调用。因为在默认的 setTimeout 方法中，this 参数会被设置为全局对象，通过 bind 方法，可以指定 setTimeout 中 this 的值。

    function LateBloomer() {
      this.petalCount = Math.ceil(Math.random() * 12) + 1;
    }

    // 在 bloom 之后格1s，调用 declare 
    LateBloomer.prototype.bloom = function() {
      window.setTimeout(this.declare.bind(this), 1000);
    };

    LateBloomer.prototype.declare = function() {
      console.log('I am a beautiful flower with ' +
        this.petalCount + ' petals!');
    };

    var flower = new LateBloomer();
    flower.bloom();  // after 1 second, triggers the 'declare' method

#### 3.4.4 Bound functions used as constructors 把绑定函数当成构造函数一样使用

    TIP：这节介绍的功能属于 bind 方法中的边缘使用，这些使用并不是最佳解决方案，不应该出现在生产环境中。这里简单贴写代码。

    function Point(x, y) {
      this.x = x;
      this.y = y;
    }

    Point.prototype.toString = function() { 
      return this.x + ',' + this.y; 
    };

    var p = new Point(1, 2);
    p.toString(); // '1,2'

    var emptyObj = {};
    var YAxisPoint = Point.bind(emptyObj, 0/*x*/);
    // not supported in the polyfill below,
    // works fine with native bind:
    var YAxisPoint = Point.bind(null, 0/*x*/);

    var axisPoint = new YAxisPoint(5);
    axisPoint.toString(); // '0,5'

    axisPoint instanceof Point; // true
    axisPoint instanceof YAxisPoint; // true
    new Point(17, 42) instanceof YAxisPoint; // true

#### 3.4.5 Creating shortcuts 创建快捷方式

当你想创建一个有特殊 this 值的函数快捷方式时，bind 是非常有用的。通过下面的对比就可以知道， bind 创建的方式更加高效。

    // 不使用 bind 的情况
    var slice = Array.prototype.slice;

    // ...

    slice.apply(arguments);

    ////////////////////////////////////////////

    // 使用 bind 的情况
    var unboundSlice = Array.prototype.slice;
    var slice = Function.prototype.apply.bind(unboundSlice);

    // ...

    slice(arguments);
    
### 3.5 Polyfill

因为 bind 是  ECMA-262 的标准，所以一部分浏览器是不支持的。下面则是让浏览器支持的兼容代码。

    if (!Function.prototype.bind) {
      Function.prototype.bind = function(oThis) {
        if (typeof this !== 'function') {
          // closest thing possible to the ECMAScript 5
          // internal IsCallable function
          throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
        }

        var aArgs   = Array.prototype.slice.call(arguments, 1),
            fToBind = this,
            fNOP    = function() {},
            fBound  = function() {
              return fToBind.apply(this instanceof fNOP
                     ? this
                     : oThis,
                     aArgs.concat(Array.prototype.slice.call(arguments)));
            };

        if (this.prototype) {
          // native functions don't have a prototype
          fNOP.prototype = this.prototype; 
        }
        fBound.prototype = new fNOP();

        return fBound;
      };
    }

### 总结

到这里，call、apply 和 bind 的介绍就完了。那么，这三者有什么区别和联系呢？

call 能做的事情，apply 也可以做。但 apply 能做的事情，call 不一定行。然后，apply 可以对内置函数进行巧妙的调用。这两个方法，并没有哪一个是多余的。

那么 call/apply 和 bind 之间的区别呢？既然有了 call/apply，为什么 ES5 还是增加了 bind ？我们什么时候才应该用 bind ？

**区别**：call/apply 是立刻调用函数的，bind 是返回一个函数，然后可以在其他时候调用。

    var person = {  
      name: "James Smith",
      hello: function(thing) {
        console.log(this.name + " says hello " + thing);
      }
    }
    // call
    person.hello.call(person, "world"); // output: James Smith says hello world

    // bind
    var helloFunc = person.hello.bind(person);
    helloFunc("world");  // output: James Smith says hello world

**使用**：bind 一般用在异步调用和事件，下面来看个例子。

    function MyObject(element) {
        this.elm = element;

        element.addEventListener('click', this.log, false);
    };

    MyObject.prototype.log = function(e) {
         var t = this; 
         console.log(this);
    };

    var submit = document.getElementById("submit");

    var _obj = new MyObject(submit);

上面例子，在调用的时候，log 方法里面的 this 是什么？想一下。

    // log this: <input type="button" id="submit" class="button" value="点击">

是的，log 里面的 this 就是绑定事件的元素。但我们其实想要的是 MyObject 对象，那怎么办呢？加个 bind 就可以了。

    function MyObject(element) {
        this.elm = element;

        element.addEventListener('click', this.log.bind(this), false);
    };

    MyObject.prototype.log = function(e) {
         var t = this; 
         console.log(this); // MyObject {elm: input#submit.button}
    };

    var submit = document.getElementById("submit");

    var myO = new MyObject(submit);

嗯，看懂了吧～这三个函数认真琢磨下还是很有用的，快来使用吧。

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call
[2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind

