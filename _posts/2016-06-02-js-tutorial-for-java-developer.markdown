---
layout: post
title:  "写给Java程序员的JavaScript教程"
date:   2016-06-02 12:51:40 +0800
categories: javascript
---
## 1.基本数据类型

- `undefined` 当声明的变量未初始化时，该变量的默认值是 undefined;
- `number` 既可以表示 32 位的整数，还可以表示 64 位的浮点数;
- `string` 字符串字面量是由双引号（"）或单引号（'）声明;
- `boolean` 它有两个值 true 和 false;
- `null` Null 类型, 表示尚未存在的对象，如果函数或方法要返回的是对象，那么找不到该对象时，返回的通常是 null;

## 2、变量声明

	var a ＝ "test";

- 弱类型，不需要指明具体类型
- `var` 关键字表示局部变量，去掉 var 关键字表示全局变量
- `;` 可以省略

## 3、基本逻辑语句

if、while、for、switch、break、continue与Java用法一致

## 4、函数与对象
### 函数

	function functionName(arg0, arg1, ... argN) {
	  statements
	}
函数也是一种对象，对应的类是`Function`。

### 对象
- 声明和实例化
	
	var o = new Object();

- 作用域

一个类的所有属性和方法都是公开的，可以直接访问；约定使用`下划线`开始的变量和方法是私有的；
没有静态变量；

- this关键字

指向调用该方法的对象，如果不用this，就会被看作是局部变量或全局变量。然后该函数将查找局部或全局变量，找不到该函数将在警告中显示 "null"；

- 属性与方法

动态添加，所有属性和方法都需要在实例化对象后动态添加；

- 继承

使用原型链来实现继承，prototype 对象是个模板，要实例化的对象都以这个模板为基础，prototype 的任何属性和方法都被传递给类的所有实例。原型链利用这种功能来实现继承机制。
	
	function ClassA(sColor) {
	    this.color = sColor;
	}
	
	ClassA.prototype.sayColor = function () {
	    alert(this.color);
	};
	
	function ClassB(sColor, sName) {
	    ClassA.call(this, sColor);
	    this.name = sName;
	}
	
	ClassB.prototype = new ClassA();
	
	ClassB.prototype.sayName = function () {
	    alert(this.name);
	};

## ECMAScript 6
ECMAScript和JavaScript的关系是，前者是后者的规范，后者是前者的一种实现（另外的ECMAScript方言还有Jscript和ActionScript）;
Node.js是JavaScript语言的服务器运行环境，对ES6的支持度比浏览器更高;

### 变量声明
用let声明变量，变量作用域限制在代码块内

const声明常量，用法和java的final一致，变量赋值后不可更改

### 变量解构赋值

解构赋值特性可以从数组或者对象中提取相应的值，例如

	// 数组解构 a=1, b=2, c=3
	var [a, b, c] = [1, 2, 3];
	
	// 对象解构 a="aaa", b="bbb"
	var {a, b} = { a: "aaa", b: "bbb"};
	
	// 函数参数解构
	function add([x, y]) {
		return x + y;
	}
	add([1, 2]);

### 类 Class
- 定义：使用class关键字定义类，比原先基于原型的写法更直观
和清晰。定义类方法**不需要** `function` 关键字，方法之间不需要逗号分隔。

	之前的写法：

		function Point(x,y){
		  this.x = x;
		  this.y = y;
		}
		
		Point.prototype.toString = function () {
		  return '(' + this.x + ', ' + this.y + ')';
		};
	
	ES6的写法：

		class Point {
		  constructor(x, y) {
		    this.x = x;
		    this.y = y;
		  }
		
		  toString() {
		    return '(' + this.x + ', ' + this.y + ')';
		  }
		}

- 继承：通过`extends`关键字实现继承

	class ColorPoint extends Point {
	  constructor(x, y, color) {
	    super(x, y); // 调用父类的constructor(x, y)
	    this.color = color;
	  }
	
	  toString() {
	    return this.color + ' ' + super.toString(); // 调用父类的toString()
	  }
	}

- 静态方法：方法之前添加static，通过类调用

		class Foo {
		  static classMethod() {
		    return 'hello';
		  }
		}
		
		Foo.classMethod() // 'hello'

### 模块 Module
主要由两个命令构成：`export`和`import`，`export`用于向外提供借口，`import`用于引入接口。

	import { func1, func2 } from 'moduleA';
	
	import React from 'react';
	const Breadcrumbs = React.createClass({
	  render() {
	    return <nav />;
	  }
	});

	
### 数组扩展

`Array.from()` 可以用于将可遍历对象转为数组类型，如：
	
	Array.from('hello')
	Array.from(new Set(['a', 'b']))
	
还可以作用于类数组对象（包含length属性的对象），如函数的arguments对象。

	function foo() {
		var args = Array.from(arguments);
	}

Array.from 可以传人地二个参数，用于对数组对象做变换

	Array.from(arrayLike, x => x * x);
	


## Reference
- [http://www.w3school.com.cn/js/index_pro.asp](http://www.w3school.com.cn/js/index_pro.asp)
- [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/)
