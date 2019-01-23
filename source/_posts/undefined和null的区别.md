title: undefined和null的区别
date: 2016-05-19 16:37:31
categories: js
tags: js
---
js中的`undefined`和`null`是何等的相似，几乎是一样的，所以要说一下细微的区别，先动手操作一下，在浏览器Console中执行：<!-- more -->
```
undefined == null //返回值 true
```
这说明一定情况下undefined和null可以转化成false。执行
```
Number(undefined)  //返回值 NAN

Number(null)  //返回值 0
```
这也是说明undefined和null的区别所在。执行
```
typeof(undefined) //返回值 "undefined"

typeof(null)  //返回值 "object"
```
但是以上的区别在实际使用中又不是很明显。
**undefined**表示"缺少值"，表示此处应该有一个值，但是还没有定义，这种情况下使用`undefined`，常用方法：
>1. 变量被声明了，但没有赋值时，就等于undefined
>2. 调用函数时，应该提供的参数没有提供，该参数等于undefined
>3. 对象没有赋值的属性，该属性的值为undefined
>4. 函数没有返回值，默认返回的是undefined

```
var a;
a // undefined

function test(b){
	console.log(b)
}
test(); //undefined

var newObj = new Object();
	newObj.hello;
// undefined

var c = test();
c //undefined
```

**null**表示"没有对象"，表示此处不应该有值，这种情况下使用`null`，常用方法：
>1. 作为函数的参数，表示该函数的参数不是对象
>2. 作为对象原型链的终点

```
Object.getPrototypeOf(Object.prototype)
// null
```
就酱~