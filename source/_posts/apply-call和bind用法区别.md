title: 'apply、call和bind用法区别'
date: 2016-06-10 17:48:22
categories: js
tags: js
---
每个函数中都包含两个非继承而来的方法：`apply()`, `call`。这两个方法的用途都是在特定的作用域下调用函数，实际上等于设置函数体内this对象的值。首先，`apply()`方法接收两个参数：一个是在其中运行函数的作用域，另一个是参数数组。其中，第二个参数可以是Array的实例，也可以是arguments对象，例如：<!--more-->
```
function sum (num1, num2){
	return num1 + num2;
}

function sumCall1(num1, num2){
	return sum.apply(this, arguments);  //传入arguments对象
}

function sumCall2(num1, num2){
	return sum.apply(this, [num1,num2]);  //传入数组
}

sumCall1(1,3);  // 4
sumCall2(1,3);  // 4
```
上面例子中，sumCall1()在执行sum()函数时传入了this作为this值（因为是在全局作用域中调用的，所以传入的就是window对象）和arguments对象，然而sumCall2()同样也调用sum()函数，但它传入的则是this和一个参数数组，都返回同样的结果。
>在严格模式下，未指定环境对象而调用函数，则this值不会转成window。除非明确把函数添加到某个对象或者调用`apply()`或`call()`，否则this值将是undefined。

`call()`方法与`apply()`方法的作用相同，它们的区别在于接收参数的方式不同，对于`call()`方法而言，第一个参数是this不变，变化的是其余参数都是直接传递给函数，也就是说在使用`call()`时，传递函数的参数要一一列举出来，看下面的例子🌰：
```
function sum(num1, num2){
	return num1 + num2;
}

function callSum(num1, num2){
	return sum.call(this, num1,num2);
}

callSum(1，3) // 4
```
在使用`call()`方法的情况下，callSum要传入每一个参数值，结果与使用`apply()`并没有什么不同。至于是使用`apply()`还是`call()`，完全取决于你采用哪种给函数传递参数的方式最方便。如果你打算直接传入arguments对象，或者包含函数中先接收到的也是一个数组，那么使用`apply()`肯定很方便；否则，选择`call()`更合适，（再不给函数传递参数的情况下，使用哪个方法都无所谓。）

事实上，传递参数并非`apply()`和`call()`真正用武之地，它们最强大的地方是能够扩充函数的赖以运行的作用域，看下面例子🌰：
```
window.color = "red";
var o.color = "blue";

function sayColor(){
	alert(this.color);
}

sayColor();

sayColor.call(this);	//red
sayColor.call(window);	//red
sayColor.call(o);		//blue
```
使用`apply()`或`call()`来扩充作用域的最大好处就是对象不需要与方法有任何耦合关系。在ES5中还定义了一个方法`bind()`。这个方法会创建一个函数的实例，其this值会被绑定到传给`bind()`函数的值。再来个例子🌰。
```
window.color = "red";
var o.color = "blue";

function sayColor(){
	alert(this.color);
}

var objSayColor = sayColor.bind(o);
objSayColor();	//blue
```
在这里，`sayColor()`调用`bind()`并传入对象o，创建了`objSayColor()`函数，`objSayColor()`函数this值等于o，因此即使是在全局作用域中调用这个函数，也会看到"blue"。

补充：
传入的第一个参数为`null`，函数体内的this会默认指向宿主对象，在浏览器中，如果使用严格模式，则还为null。
函数是在非严格模式下，指定为 `null` 或 `undefined` 的 this 值会被替换成全局对象，
就先说这么多~