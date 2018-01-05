---
layout: post
title: 你不知道的 JavaScript
date: 2017-03-09 22:23:55
description: JavaScript 排坑
img: jsbanana.png
tags: [javascript]
---

#### 1.`typeof null === 'object'?`
众所周知，在 JavaScript 中一共有六种主要简单基本类型：
* string
* number
* boolean
* null
* undefined
* object

但是，null 有时会被当作一种对象类型，这主要来自于 JavaScript 本身的一个 bug，即对 null 执行 typeof null 时返回字符串 object。
原理是这样的，不同的类型在底层都表示为二进制，在 JavaScript 中二进制前三位都为0的话会被判为 object 类型， null 的二进制表示全是0，
所以执行 typeof 时会返回 "object" 。实际上， null 本身是基本类型。

#### 2.对象的属性描述符
我们来看如下这段代码：

```javascript
var myObject = {
  a: 2
};

myObject.a = 3;
myObject.a; //3

Object.defineProperty(myObject, "a", {
  value: 4, //值
  writable: true, //可写
  configurable: false, //不可配置！
  enumerable: true //可枚举
});

myObject.a; //4
myObject.a = 5;
myObject.a; //5

Object.defineProperty(myObject, "a", {
  value: 6, 
  writable: true, 
  configurable: true, 
  enumerable: true 
}); //TypeError
```
最后一个 defineProperty(..) 会产生一个 TypeError 错误，不管是不是处于严格模式，尝试修改一个不可配置的属性描述符都会出错。
注意：**如你所见，把 configurable 修改成 false 是单向操作，无法撤销！但是有一个例外，即便属性是 configurable:false, 我们还是
可以把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。**

> 未完待续，本篇文章将长期更新...