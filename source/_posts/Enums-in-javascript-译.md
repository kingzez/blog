title: 'Javascript中的枚举 [译]'
date: 2016-05-23 10:35:39
categories: js
tags: js
---
** *Javascript中的枚举* ，**你想这样定义枚举：<!-- more -->
```
var SizeEnum = {
  SMALL: 1,
  MEDIUM: 2,
  LARGE: 3,
};
```
然后这样用它：
```
var mySize = SizeEnum.SMALL;
```
如果你想把枚举对应的值加上属性，你可以给它们加一个额外的对象：
```
var SizeEnum = {
  SMALL: 1,
  MEDIUM: 2,
  LARGE: 3,
  properties: {
    1: {name: "small", value: 1, code: "S"},
    2: {name: "medium", value: 2, code: "M"},
    3: {name: "large", value: 3, code: "L"}
  }
};
```
然后像这样用
```
var mySize = SizeEnum.MEDIUM;
var myCode = SizeEnum.properties[mySize].code; // myCode == "M"
```
#### 背景
上面描述的是对JavaScript枚举思考很长时间得出的结果。它尝试最好的结合以原语为枚举值（序列化）和以对象作为值（允许值对应属性），进一步阅读了解我是怎么实现这一方法。

#### 重新理解枚举
我最近偶然发现自己在StackOverflow已经回答了几年的问题，读过一些评论之后思考了一下觉得这是一个很值得话题。

*那么评论中都是那些问题呢？*

**在JavaScript中写枚举最好的方法是？**
首先，在回答问题之前，先了解一下什么是枚举？在JavaScript中写意着什么。让我们先看一下枚举的定义：
**枚举是什么?**
>在数学和计算机科学理论中，一个集的枚举是列出某些有穷序列集的所有成员的程序，或者是一种特定类型对象的计数。这两种类型经常（但不总是）重叠。枚举是一个被命名的整型常数的集合
*–维基百科: 枚举*

一个很好的例子：
```
enum WeekDay = {MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY};
```
总结：枚举是一组预定义的常量中的一个制约变量。在上面的例子中WeekDay是一个枚举值，MONDAY、TUESDAY等是集合中的常量，也可称为枚举成员，如果我们声明一个变量：
```
WeekDay payDay;
```
我们可以把它分配给MONDAY，TUESDAY等任何一个常量，直到包括SUNDAY，但不是其他的什么，像12、`labour day`。
…这给我们带来一个问题。
**它不能在JavaScript中实现**
javascript是弱类型语言，这意味着不需要事先声明一个特定类型的变量。在Java（强类型）中你可能这样写：
```
int i; // declares a variable named i which may hold integer values.
```
如果之后你要给它分配一个字符串，你需要这样：
```
i = "Hello World";
```
但是编译器会返回错误，中断生成文件
不像Javascript中:
```
var i;
i = 10;
i = "Hello World";
i = 3.1415;
i = true;
i = ['my', 'array'];
i = {look: 'at', my: 'object'};
```
正如你所看到，我们声明了一个变量（通过关键字var）,但它在运行时类型是不受限制的。我们可以给他分配任何值。维基百科的措辞有点尴尬，但是你读到这行
>A variable that has been declared as having an enumerated type can be assigned any of the enumerators as a value.

它表明，任何的枚举数都可被分配，但是什么都没有，所以它限制了可以分配给变量的值。在JavaScript中我们只是不能这样做，因此我们不能写真正的枚举，我们可以模仿C语言中提供的一些方便的方法，但要记住，我们只模仿它提供的语法糖。

**在javascript中写枚举**
我是从强类型的java中转过来的，但也很长时间没有写枚举了，因此Java的编程给我带来一些不同的解决方法来模仿写出枚举。

*常数，常常是i、c、w。命名约定*
只列出你使用类顶部的常数
```
public static final int DAYS_MONDAY = 0;
public static final int DAYS_TUESDAY = 1;
// ..
public static final int DAYS_SUNDAY = 6;
```
*简单常量类*
像上面简单的例子，把常量放到专用的类中用于复用：
```
public class DaysEnum {
  public static final int MONDAY = 0;
  public static final int TUESDAY = 1;
  // ..
  public static final int SUNDAY = 6;
}
```
*常数实例类*

Java的枚举促使了js枚举的仿制，它使用一个专用的私有构造函数类（除了类本身没有被实例化）和使用实例本身作为枚举值：
```
public class DaysEnum {
  private DaysEnum() {}
  public static final DaysEnum MONDAY = new DaysEnum();
  public static final DaysEnum TUESDAY = new DaysEnum();
  // ..
  public static final DaysEnum SUNDAY = new DaysEnum();
}
```
另外，使用枚举实例的枚举数是一种优雅的方式，这使得这样模拟枚举类型的安全，与其他两个变量比，枚举的变量将是整数：
```
int payDay = DAYS_FRIDAY; // variation 1
int payDay = DaysEnum.FRIDAY; // variation 2
```
它仍然有可能指定一个完全错误的值，如128这样的枚举。相比之下，第三的变异实际上限制可以被分配到枚举列出枚举成员的值：
```
DaysEnum payDay = DaysEnum.FRIDAY; // ok
DaysEnum payDay = 128; // compiler error
```
作为新增加的亮点，第三枚举模式允许我们添加额外的字段，例如把一天加上名字，甚至方法（isWeekendDay()为例）的枚举。因此，从Java中了解的这些模式，我在StackOverflow上建议JavaScript使用第三枚举的方法来写。
目前仍是在StackOverFlow上得到的星最多的，因此我要解释一下为什么，然后告诉你我个人认为在JavaScript中获得最大的收益就是要尽可能的少写枚举。

***那么替代这种方法的又是什么呢？ 这样的方法又有什么错误呢？***

第三枚举在JavaScript中是这样：
```
var DaysEnum = {
  MONDAY: {}, // optionally you can give the object properties and methods
  TUESDAY: {},
  // ..
  SUNDAY: {}
};
```
但是，我现在不再推荐这种风格，不要再使用它了。
***为什么不呢?***
因为在评论中有人指出关于线程的问题，当数据序列化的时候这将出现一些问题，为了理解为什么这样，让我们来看一看当我们把DaysEnum中的一个枚举值作为一个对象的字段会发生什么：
```
var myObject = {
  payDay: DaysEnum.MONDAY
};

var yesterday = DaysEnum.THURSDAY, 
    today = DaysEnum.MONDAY;
if (yesterday == myObject.payDay)
  alert("Yesterday was pay day... but not today...");
else if (today == myObject.payDay)
  alert("Today is pay day! Yippie!!!");
else
  alert("Neither yesterday nor today are pay days... I'm broke!");
```
到目前为止很好，它会alert出我们预期的"Today is pay day! Yippie!!!"(今天是周一)，但是我们将`myObject`序列化转成JSON,再将它反序列化：
```
var serialized = JSON.stringify(myObject);
  alert("serialized myObject: " + serialized);
var deserializedObject = JSON.parse(serialized);
if (yesterday == deserializedObject.payDay)
  alert("Yesterday was pay day... but not today...");
else if (today == deserializedObject.payDay)
  alert("Today is pay day! Yippie!!!");
else
  alert("Neither yesterday nor today are pay days... I'm broke!");
```
这个结果是"Neither yesterday nor today are pay days... I'm broke!"原因是依靠反序列化`JSON.parse`创建一个新的对象作为payDay的值，这个新对象不同于`DaysEnum.FRIDAY`因此在比较之后会错误。

我认为这样解决问题不是一个很好的方法，能系列化和反序列化枚举是很重要的，而不能忽视的。这就是为什么我建议不要使用这种模式，相反的是用第二枚举解决问题就不会出现这样的问题：
```
var DaysEnum = {
  MONDAY: "monday",
  TUESDAY: "tuesday",
  // ..
  SUNDAY: "sunday"
}; 
```
（或者你可以用数字代替字符串的值，都可以达到同样的效果），让我们检查一下这个做系列化的安全性：
```
var myObject = {
  payDay: DaysEnum.FRIDAY
};

var serialized = JSON.stringify(myObject);
  alert("serialized myObject: " + serialized);
var deserializedObject = JSON.parse(serialized);
if (yesterday == deserializedObject.payDay)
  alert("Yesterday was pay day... but not today...");
else if (today == deserializedObject.payDay)
  alert("Today is pay day! Yippie!!!");
else
  alert("Neither yesterday not today are pay days... I'm broke!");
```
正如预期的那样，结果是"Today is pay day! Yippie!!!"。但是我绝得给枚举的添加字段和方法真的很棒，我们可以这样做：
```
var SizeEnum = {
  SMALL: {name: "small", value: 1, code: "S"},
  MEDIUM: {name: "medium", value: 2, code: "M"},
  LARGE: {name: "large", value: 3, code: "L"},
};
```
但是可以说这不需要序列化和反序列化，我们只需要给额外的对象增加一个属性W:
```
var SizeEnum = {
  SMALL: 1,
  MEDIUM: 2,
  LARGE: 3,
  properties: {
    1: {name: "small", value: 1, code: "S"},
    2: {name: "medium", value: 2, code: "M"},
    3: {name: "large", value: 3, code: "L"}
  }
};
```
这样我们能够取到枚举数的属性值，像这样：
```
var mySize = SizeEnum.MEDIUM;
var myCode = SizeEnum.properties[mySize].code; // myCode == "M"
```
……不可否认优雅的方式。我想说的是，对于一个枚举值的序列化/反序列化的能力远比在实例上属性更重要。

***Object.freeze***
在类型安全的语言中，枚举的这些类型都是不变的，值得集合不改变也不做自身的常量值，在JavaScript中，任何时候我们都可以覆盖赋值所有的常数，或添加新的属性，如果你想避免这种情况，可以看一下[Objcet.freeze](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

Object.freeze() 方法可以冻结一个对象。冻结对象是指那些不能添加新的属性，不能修改已有属性的值，不能删除已有属性，以及不能修改已有属性的可枚举性、可配置性、可写性的对象。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。

听起来还真是一个很需要的方法，是吗~？

因为不是所有的浏览器都支持，你应该在使用前测试是否可用：
```
if (Object.freeze)
  Object.freeze(DaysEnum);
```
好了，就到这~JavaScript中的枚举，写代码去！

第一次翻译外文，水平有限，如有误，请批正。

