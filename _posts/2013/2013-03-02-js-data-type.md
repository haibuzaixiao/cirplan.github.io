---
layout: post
category : JS
title: javascript数据类型
tags : [js]
---
{% include JB/setup %}

5种数据类型：`undefined；null；boolean；number；string`；
特殊：`object`；

`typeof`（操作符）：检测数据类型。

* 返回字符串；
* 返回object，是对象 或者 null；

`undefined`：变量没初始化时；不用把值显式设置为undefined。

`null`：空对象指针；要把值显示设置为null。

`boolean`：ture || false; 不是 Ture || Flase；

* 一般情况：ture=1；false=0；
* string：非空为ture；
* number：非零数值（包括无穷大）为ture；
* object：任何对象为ture；
* undefined：n/a（不适用）为ture；

`number`：

* 八进制--第一位为0，数字:0~7；
* 十六进制--ox；数字0~9及A~F;字母可大写小写；
* 浮点数：小数点后没数值，ex：1.解析为1；整数，ex：10.0解析为10；浮点数值精确度不大好，ex：0.1+0.2=0.3000000000004；
* 数值范围：正无穷：Infinity；负无穷：-Infinity； --检测函数（位于最大最小值之间为true）：`isFinite()`；
* NaN：非数值，与任何值都不相等，包括自身；--检测函数（接收一个参数，尝试把参数转换成数值，确定参数是否“不是数值”）：`isNaN()`；
* 数值转换：三函数--`Number()`，`parseInt()`和`parseFloat()`；

`Number()`:

* null->0;
* undefined->NaN；
* 字符串->空的:0;其他格式:NaN;
* 对象：调用valueOf(),若转换为NaN，则调用对象的toString()；

`parseInt()`：

* 规则--忽略前面的空字符串，直到找到第一个非空字符串，若不是数字或符号，返回NaN;
* 字符串为空,返回NaN；
* 解析到非数字字符串，ex：1234.12返回1234；
* 0x>>16进制；0>>8进制；
* 由于E3，E5的区别，转换时可以再增加一个参数：ex：parseInt（“10”，2）；

`parseFloat()`：第一个小数点有用，第二个小数点没有；只解析十进制;

`string`：字符字面量，length包括字符字面量；字符串创建了就不能改变；要改变，先销毁；

* `toString()`--null和undefined没有这种方法；参数，输出数值的基数，ex：toString(2)--二进制；
* `String()`--不知道转换值是null或undefined时；null->"null",undefined->"undeifined"
