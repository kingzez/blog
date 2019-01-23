title: 深入浅出this的理解
date: 2017-04-11 10:45:18
categories: js
tags: this
---

`this`在js中一直是谜一样的存在着，在面试中也是经常会被问道，接下来总结下我所理解的`this`。
<!-- more -->首先引入[秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/#function.this)中对this的工作原理，文中指出有五种情况，分别是：

- 全局范围内

```javascript
this;	//在全局范围内使用`this`，它将会指向全局对象
```

- 函数调用

```javascript
foo();	//this指向全局对象
```

- 方法调用

```javascript
test.foo();	//this指向test对象
```

- 调用构造函数

```javascript
new foo();	//函数与new一块使用即构造函数，this指向新创建的对象
```

- 显式的设置this

```javascript
function foo(a, b, c) {}
var bar = {};
foo.apply(bar, [1, 2, 3]);	//this被设置成bar
foo.call(bar, 1, 2, 3);	//this被设置成bar
```

#### 从函数调用理解this
有这样一道题

```javascript
var obj = {
	func: function(){
		console.log(this);
	}
}
var foo = obj.func;
obj.func() //this是obj
foo() //this是window
```

解释函数结果为什么不一样？

这道题可以从函数调用来理解，从我看到的博文中有这样的解释，秘密花园中的函数调用，方法调用，显式的设置this都属于函数调用，相当于函数调用的三种方式，可以写成

```javascript
foo(params);
obj.child.foo(params);
foo.call(context, params); //apply同理
```

而且前两种都可以写成第三种形式

```javascript
foo(params); =>> foo.call(undefined, params);
obj.child.foo(); =>> obj.child.foo.call(obj.child, params);
```

所以说函数调用都是`foo.call(context, params)`的变体，即函数调用只有一种。这样`this`的值就是文中的`context`，`this`就是你函数调用时传入的`context`。

举例代码中：

```javascript
function foo() {
	console.log(this);
}
foo();
```

当你无法确认`this`指代的值时，转化成`call`形式会更好理解

```javascript
function foo() {
	console.log(this);
}
foo.call(undefined); //or foo.call();
```

函数执行结果为`window`，在浏览器中有一条规定，传入`context`为`null`或`undefined`时，则为全局对象（window对象）；严格模式下`context`为`undefined`。

举例代码中：

```javascript
var obj = {
	func: function foo() {
		console.log(this);
	}
}
obj.foo();
```

转化为`call`形式为

```javascript
var obj = {
	func: function foo() {
		console.log(this);
	}
}
obj.foo.call(obj);
```

很明显，`this`指代obj。

#### Event Handler 中的this

```javascript
btn.addEventListener('click' ,function handler(){
  console.log(this) // 请问这里的 this 是什么
})
```

handler 中的`this`是什么？用上面的方法好像没法转化了，因为不知道`addEventListener`内源码实现，查看文档[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener#The_value_of_this_within_the_handler)
>通常来说this的值是触发事件的元素的引用，这种特性在多个相似的元素使用同一个通用事件监听器时非常让人满意。

>当使用 addEventListener() 为一个元素注册事件的时候，句柄里的 this 值是该元素的引用。其与传递给句柄的 event 参数的 currentTarget 属性的值一样。

所以可以假设浏览器addEventListener是这样实现的

```javascript
// 当事件被触发时
handler.call(event.currentTarget, event) // this => event.currentTarget
```

以上就是对`this`的理解。