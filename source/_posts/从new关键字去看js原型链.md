---
title: 从new关键字去看js原型链
date: 2017-09-28 19:39:19
tags: js
categories: 前端
thumbnail: 
- http://owznda5oi.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-10-05%20%E4%B8%8A%E5%8D%8810.40.55.png
---
涉及到稍微深入点的js，就不得不谈到js的new关键字。看了很多篇文章，发现这个new就是个糖，他的存在帮助前端开发者节省代码，让大家写出来的代码更加可读而美丽。

### 用new和不用new去完成同样的事情
假设我们要批量去生产一批对象，这批对象有要都有abc与cba属性，同时每个对象都有各自的id，怎么办呢，下面我要分别用new与不用new去实现这个需求。

new
``` javascript
function a() {
   this.id = 1234567890;
}
a.prototype = {
   abc: 'abc',
   cba: 'cba'
}
var b = new a();
```
不用new
``` javascript
a.prototype = {
   abc: 'abc',
   cba: 'cba'
}
function b(){
   var new = {};
   new.__proto__ = a.protoype;
   new.id = 1234567890;
   return new;
}
```
就这个简单的需求来看，new就是帮我们省了几行代码……

1、new帮我创建一个临时对象。

2、new帮我绑定原型。

3、new帮我们return

4、统一叫做prototype

也就是说使用new我们可以少做4件事情：
> * 不用创建临时对象，因为 new 会帮你做（你使用「this」就可以访问到临时对象）；
> * 不用绑定原型，因为 new 会帮你做（new 为了知道原型在哪，所以指定原型的名字为 prototype）；
> * 不用 return 临时对象，因为 new 会帮你做；
> * 不要给原型想名字了，因为 new 指定名字为 prototype。

通过new操作符调用的函数就是构造函数。由构造函数构造的对象，其[[prototype]]指向其构造函数的prototype属性指向的对象。每个函数都有一个prototype属性，其所指向的对象带有constructor属性，这一属性指向函数自身。（在本例中，b的[[prototype]]指向a.prototype）

### __proto__与prototype
我们新建一个对象，再打印出来打，然后用chrome的f12看看：
``` javascript
var c = {
   a: 1,
   b: 2,
   c: 3,
   h: 1,
   i: 1,
   j: 1,
   k:1
}
console.log(c);
//chrome 下
```
![chrom](http://owznda5oi.bkt.clouddn.com/v2-cf578dbe50fd529abb868988bd9f530e_b.png)

真**神奇哈，咱们新建这个对象天生就有一个__proto__属性，该属性指向的是这个对象的原型。

再来看网上找到的另一个复杂一些的例子：

``` javascript
var Person = function () { };
		Person.prototype.Say = function () {
		alert("Person say");
	}
	Person.prototype.Salary = 50000;
 
	var Programmer = function () { };
	Programmer.prototype = new Person();
	Programmer.prototype.WriteCode = function () {
		alert("programmer writes code");
	};
 
	Programmer.prototype.Salary = 500;
 
	var p = new Programmer();
	p.Say();
	p.WriteCode();
	alert(p.Salary);
//chrome 下
```
var p=new Programmer()可以得出p.__proto__=Programmer.prototype;

p1.__proto__=Person.prototype;

Programmer.prototype.__proto__=Person.prototype;

由根据上面得到p.__proto__=Programmer.prototype。可以得到p.__proto__.__proto__=Person.prototype。

好，算清楚了之后我们来看上面的结果,p.Say()。由于p没有Say这个属性，于是去p.__proto__，也就是 Programmer.prototype，也就是p1中去找，由于p1中也没有Say，那就去p.__proto__.__proto__，也就是 Person.prototype中去找，于是就找到了alert(“Person say”)的方法。