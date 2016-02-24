---
layout: post
category : js
title: Undescore.js和jQuery的类型判断
tags : [web, js]
---
{% include JB/setup %}

记得很久之前，有简单的记录过 js 中的数据类型（[点这里][1]）。那么，如何去判断一个变量的数据类型呢？

先来看看一些判断 bug ：

    typeof null // "object"
    typeof new String('abc') // "object"

    var iframeArr = new window.frames[0].Array;
    iframeArr instanceof Array // false
    
    isNaN('abc') // true

所以说，在判断类型的时候，往往不小心就产生了 bug . 那么，怎样才能准确判断数据类型？来看看 `undescore.js` 和 `jQuery` 的做法。

## 1. underscore 1.8.3

underscore 提供了一堆判断类型的函数，如下：

    - isArray
    - isObject
    - isArguments
    - isFunction
    - isString
    - isNumber
    - isFinite
    - isBoolean
    - isDate
    - isRegExp
    - isNaN
    - isNull
    - isUndefined

其核心的判断代码如下，利用 `Object.prototype.toSting`，这个也是业界公认好用的方法：

    // Add some isType methods: isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError.
    _.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name) {
        _['is' + name] = function(obj) {
            return toString.call(obj) === '[object ' + name + ']';
        };
    });

下面看看其他判断函数。

### - isArray

先判断浏览器是否支持原生ES5的 `Array.isArray` 方法，支持则直接用原生方法判断，不然利用 `Object.prototype.toSting` 来判断。

    var ObjProto = Object.prototype,
        toString = ObjProto.toString,
        nativeIsArray = Array.isArray;

    _.isArray = nativeIsArray || function(obj) {
        return toString.call(obj) === '[object Array]';
      };


### - isObject

直接用 typeof 判断。不过判断函数的时候，也是 `true` ? 这样岂不是会和 `isFunction` 有重叠？

    _.isObject = function(obj) {
        var type = typeof obj;
        return type === 'function' || type === 'object' && !!obj;
    };

    // example
    function a () {}
    console.log(_.isObject(a)); // true

### - isArguments `低于IE9 版本`

直接判断是否有 `callee` 属性。要注意的是：在 ES5 标准中，严格模式下，函数参数是没有 `callee` 属性的，详细看[这里][2]。

    _.has = function(obj, key) {
        return obj != null && hasOwnProperty.call(obj, key);
    };
    
    // 在 IE < 9 是没有 [object Arguments] 这个类型的
    if (!_.isArguments(arguments)) {
        _.isArguments = function(obj) {
            return _.has(obj, 'callee');
        };
    }

### - isFunction `优化版本`

这里会有疑问，明明直接用 typeof 判断就可以了，为什么还要加后面的 `|| false`？注释里说是因为 IE11 (& IE8) 下面的 bug。详情看[这里][3]。

    // Optimize `isFunction` if appropriate. Work around some typeof bugs in old v8,
    // IE 11 (#1621), and in Safari 8 (#1929).
    if (typeof /./ != 'function' && typeof Int8Array != 'object') {
        _.isFunction = function(obj) {
            return typeof obj == 'function' || false;
        };
    }

### - isFinite

判断一个数是否为有限，先看基本的测试：
    
    isFinite(Infinity);  // false
    isFinite(NaN);       // false
    isFinite(-Infinity); // false

    isFinite(0);         // true
    isFinite(2e64);      // true
    isFinite(null);      // true

    isFinite("0");       // true

再看 underscore.js 中的代码，排除了 NaN 情况：

    _.isFinite = function(obj) {
        return isFinite(obj) && !isNaN(parseFloat(obj));
    };

### - isNaN, - isBoolean, - isNull, - isUndefined

其他类型判断函数没什么特别的，如下。

    // Is the given value `NaN`? (NaN is the only number which does not equal itself).
    _.isNaN = function(obj) {
        return _.isNumber(obj) && obj !== +obj;
    };

    // Is a given value a boolean?
    _.isBoolean = function(obj) {
        return obj === true || obj === false || toString.call(obj) === '[object Boolean]';
    };

    // Is a given value equal to null?
    _.isNull = function(obj) {
        return obj === null;
    };

    // Is a given variable undefined?
    _.isUndefined = function(obj) {
        return obj === void 0;
    };
    
## 2. jQuery 1.11.2 

看看 jQuery 中封装的类型判断接口：

    $.type(obj)
    $.isArray(obj)
    $.isFunction(obj)
    $.isEmptyObject(obj)
    $.isPlainObject(obj)
    $.isWindow(obj)
    $.isNumeric(value)

其核心代码如下，也是采用 `Object.prototype.toSting` ：

    var class2type = {};

    var toString = class2type.toString;

    ...

    jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
        class2type[ "[object " + name + "]" ] = name.toLowerCase();
    });

    ...

    type: function( obj ) {
        if ( obj == null ) {
            return obj + "";
        }
        return typeof obj === "object" || typeof obj === "function" ?
            class2type[ toString.call(obj) ] || "object" :
            typeof obj;
    },

其中 `isArray` 和 `isFunction` 是对 `type` 函数的进一步封装。

    isFunction: function( obj ) {
        return jQuery.type(obj) === "function";
    },

    isArray: Array.isArray || function( obj ) {
        return jQuery.type(obj) === "array";
    },

下面继续看看其他函数。

### - isEmptyObject

判断是否为空对象。

    isEmptyObject: function( obj ) {
        var name;
        for ( name in obj ) {
            return false;
        }
        return true;
    },

### - isPlainObject

这是 jQuery 中特有的方法，判断一个对象是否为纯粹对象（通过 {} 或 new Object() 创建的）。

    var hasOwn = class2type.hasOwnProperty;

    ...

    isPlainObject: function( obj ) {
        var key;

        // 不能被转为true, 
        // 类型一定要是[object Object]
        // 过滤 Dom 节点和 window 对象
        if ( !obj || jQuery.type(obj) !== "object" || obj.nodeType || jQuery.isWindow( obj ) ) {
            return false;
        }

        try {
            // 没有自己的 constructor 属性的一定为 Object
            if ( obj.constructor &&
                !hasOwn.call(obj, "constructor") &&
                !hasOwn.call(obj.constructor.prototype, "isPrototypeOf") ) {
                return false;
            }
        } catch ( e ) {
            // IE8,9执行上面操作的时候，会报错，也返回 false
            return false;
        }

        // Support: IE<9
        // Handle iteration over inherited properties before own properties.
        if ( support.ownLast ) {
            for ( key in obj ) {
                return hasOwn.call( obj, key );
            }
        }

        // 在枚举属性的时候，自身的属性会先被枚举，然后再枚举继承的属性，如果枚举的最后一个属性为自身的，则为 true
        for ( key in obj ) {}

        return key === undefined || hasOwn.call( obj, key );
    },

### - isWindow

判断是否为 window 对象。利用 window = window.window 的属性。

    isWindow: function( obj ) {
        return obj != null && obj == obj.window;
    },

### - isNumeric

判断是否为数字。

    isNumeric: function( obj ) {
        // parseFloat NaNs numeric-cast false positives (null|true|false|"")
        // ...but misinterprets leading-number strings, particularly hex literals ("0x...")
        // subtraction forces infinities to NaN
        // adding 1 corrects loss of precision from parseFloat (#15100)
        return !jQuery.isArray( obj ) && (obj - parseFloat( obj ) + 1) >= 0;
    }

## 总结

js 在判断类型的道路上，曾经有很多的尝试，每个类库的判断方式都不同，但自从 `Object.prototype.toSting` 被挖掘出来后，
已经变成了公认的判断类型方法。

把 jQuery 中的 `type` 方法抽出来如下：

    var typeObj = {} ;
    "Boolean Number String Function Array Date RegExp Object Error".split(" ").forEach( function (e, i) {
        typeObj[ "[object " + e + "]" ] = e.toLowerCase();
    });

    function _typeof (obj) {
        if ( obj == null ) {
            return String( obj );
        }
        return typeof obj === "object" || typeof obj === "function" ?
            typeObj[ typeObj.toString.call(obj) ] || "object" :
            typeof obj;
    }

这样，在判断类型的时候，就可以直接用这段代码了（在没有判断函数的时候）。

### 延伸阅读

* [Sea.js 源码解析（三）][4]

[1]: /js/2013/03/02/js-data-type/
[2]: http://stackoverflow.com/questions/103598/why-was-the-arguments-callee-caller-property-deprecated-in-javascript/235760#235760
[3]: https://github.com/jashkenas/underscore/issues/1621
[4]: https://github.com/lifesinger/blog/issues/175

