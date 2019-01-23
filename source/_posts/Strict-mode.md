title: 严格模式·Strict mode
date: 2017-03-20 15:10:23
categories: js
tags: 严格模式
---

ECMAScript5的严格模式是JavaScript中的一种限制性更强的变种方式。严格模式不是一个子集：它在语义上与正常代码有着明显的差异。<!-- more -->不支持严格模式的浏览器与支持严格模式的浏览器行为上也不一样，所以不要在未经严格模式特性测试情况下使用严格模式。严格模式可以与非严格模式共存，所以脚本可以逐渐的选择性加入严格模式。

严格模式在语义上与正常的JavaScript有一些不同。首先，严格模式会将JavaScript陷阱直接变成明显的错误。其次，严格模式修正了一些引擎难以优化的错误：同样的代码有些时候严格模式下会比非严格模式下更快。第三，严格模式禁用了一些有可能在未来版本中定义的语法。

如果你想让你的JavaScript代码在严格模式下运行，可以参考[转换成严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode/Transitioning_to_strict_mode)

> 有时，你会看到符合规范的、非严格模式被称为"懒散模式"，这不是官方术语，但你应该注意到它

## 开启严格模式
严格模式可以应用到整个script标签或个别函数中。不要再封闭大括号`{}`内这样做，在这样的上下文中这么做是没有任何效果的。在`eval`代码，Function代码，时间处理属性，传入setTimeout方法的字符串和包含整个脚本的块中开启严格模式会如预期一样工作。

### 为某个script标签开启严格模式
为整个script标签开启严格模式，需要在所有语句之前放一个特定语句`"use strict";`或`'use strict';`

```javascript
// 整个语句都开启严格模式的语法
"use strict";
var v = "Hi!  I'm a strict mode script!";
```
这种语法存在陷阱，不能盲目的合并冲突代码。试想合并一个严格模式的脚本和一个非严格模式的脚本：合并后的脚本代码看起来是严格模式。反之亦然：非严格合并严格看起来是非严格的。合并均为严格模式的脚本或均为非严格模式的都没问题，只有在合并严格模式与非严格模式有可能有问题。建议按一个个函数去开启严格模式（至少在学习的过渡期要这样做）。

您也可以将整个脚本的内容用一个函数包括起来，然后在这个外部函数中使用严格模式。这样做就可以消除合并的问题，但是这就意味着您必须要在函数作用域外声明一个全局变量。

### 为某个函数开启严格模式
同样的，要给某个函数开启严格模式，得把`"use strict"`;  (或 `'use strict';` )声明一字不漏地放在函数体所有语句之前。

```javascript
function strict(){
	// 函数级别严格模式语法
  	'use strict';
  	function nested() {
  		return "And so am I!";
  	}
  	return "Hi!  I'm a strict mode function!  " + nested();
}
function notStrict() { return "I'm not strict."; }

```

## 严格模式有哪些不同
严格模式同时改变了语法及运行时行为。变化通常分为这几类：将问题直接转化为错误（如语法错误或运行时错误）, 简化了如何为给定名称的特定变量计算，简化了 `eval` 以及 `arguments`, 将写"安全“JavaScript的步骤变得更简单，以及改变了预测未来ECMAScript行为的方式。

### 将拼写错转成异常
在严格模式下, 先前被接受的拼写错误将会被认为是异常. JavaScript被设计为能使新人开发者更易于上手, 所以有时候会给本来错误操作赋予新的不报错误的语义(non-error semantics). 有时候这可以解决当前的问题, 但有时候却会给以后留下更大的问题. 严格模式则把这些失误当成错误, 以便可以发现并立即将其改正.

首先，严格模式下无法在意外创建全局变量。在普通的JavaScript里面给一个拼写错误的变量名赋值会使全局对象新增一个属性并继续“工作”（尽管后面可能出错：在现在的JavaScript中有可能）。严格模式中意外创建全局变量被抛出错误替代：
```javascript
"use strict";
                       // 假如有一个全局变量叫做mistypedVariable
mistypedVaraible = 17; // 因为变量名拼写错误
                       // 这一行代码就会抛出 ReferenceError
```

其次，严格模式会引起静默失败（silently fail，注：不报错也没有任何效果）的赋值操作抛出异常。例如NaN是一个不可写的全局变量，在正常模式下，给NaN赋值不会产生任何作用，开发者也不会受到任何错误反馈。但在严格模式下，给NaN赋值会抛出异常，任何在正常模式下引起静默失败的赋值操作（给不可写属性赋值，给只读属性（getter-only）赋值，给不可扩展对象（non-extensible object）的新属性赋值）都会抛出异常。
```javascript
"use strict";

// 给不可写属性赋值
var obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
obj1.x = 9; // 抛出TypeError错误

// 给只读属性赋值
var obj2 = { get x() { return 17; } };
obj2.x = 5; // 抛出TypeError错误

// 给不可扩展对象的新属性赋值
var fixed = {};
Object.preventExtensions(fixed);
fixed.newProp = "ohai"; // 抛出TypeError错误

```
第三, 在严格模式下, 试图删除不可删除的属性时会抛出异常(之前这种操作不会产生任何效果):
```javascript
"use strcit";
delete Object.prototype; // 抛出TypeError错误
```
第四，在Gecko版本34之前，严格模式要求一个对象内的所有属性名在对象内必须唯一。正常模式下重名属性是允许的，最后一个重名的属性决定其属性值。因为只有最后一个属性起作用，当代码要去改变属性值而不是修改最后一个重名属性的时候，复制这个对象就产生一连串的bug。在严格模式下，重名属性被认为是语法错误：
> 这个问题在ECMAScript6中已经不存在

```javascript
"use strict";
var o = {p: 1, p:2}; //语法错误
```
第五，严格模式要求函数的参数名唯一，在正常模式下，最后一个重名参数名会掩盖之前的重名参数，之前的参数仍然可以通过`arguments[i]`来访问，还不是因为完全无法访问，然而在这种隐藏毫无意义而且可能是意料之外的（比如它可能本来是打错的），所以在严格模式下重名参数被认为是语法错误：
```javascript
function sum(a, a, c){ // !!! 语法错误
  "use strict";
  return a + b + c; // 代码运行到这里会出错
}
```
第六, 严格模式禁止八进制数字语法. ECMAScript并不包含八进制语法, 但所有的浏览器都支持这种以零(0)开头的八进制语法: 0644 === 420 还有 "\045" === "%".在ECMAScript 6中支持为一个数字加"0o"的前缀来表示八进制数.
```javascript
var  a = 0o10; //ES6: 八进制
```
有些新手开发者认为数字的前导零没有语法意义, 所以他们会用作对齐措施 — 但其实这会改变数字的意义! 八进制语法很少有用并且可能会错误使用, 所以严格模式下八进制语法会引起语法错误:
```javascript
"use strict";
var sum = 015 + // !!! 语法错误
          197 +
          142;
```
第七，ECMAScript 6中的严格模式禁止设置primitive值的属性.不采用严格模式,设置属性将会简单忽略(no-op),采用严格模式,将抛出TypeError错误
```javascript
(function() {
	"use strict";
	false.true = "";		//TypeError
	(14).sailing = "home";		//TypeError
	"with".you = "far away";	//TypeError
})();
```
### 简化变量的使用
严格模式简化了代码中变量名字映射到变量定义的方式. 很多编译器的优化是依赖存储变量X位置的能力：这对全面优化JavaScript代码至关重要. JavaScript有些情况会使得代码中名字到变量定义的基本映射只在运行时才产生. 严格模式移除了大多数这种情况的发生, 所以编译器可以更好的优化严格模式的代码.

首先, 严格模式禁用 with.  with 所引起的问题是块内的任何名称可以映射(map)到with传进来的对象的属性, 也可以映射到包围这个块的作用域内的变量(甚至是全局变量), 这一切都是在运行时决定的: 在代码运行之前是无法得知的. 严格模式下, 使用 with 会引起语法错误, 所以就不会存在 with 块内的变量在运行是才决定引用到哪里的情况了:

```javascript
"use strict";
var x = 17;
with (obj) // !!! 语法错误
{
  // 如果没有开启严格模式，with中的这个x会指向with上面的那个x，还是obj.x？
  // 如果不运行代码，我们无法知道，因此，这种代码让引擎无法进行优化，速度也就会变慢。
  x;
}
```
一种取代 `with` 的简单方法是，将目标对象赋给一个短命名变量，然后访问这个变量上的相应属性。

第二, 严格模式下的 eval 不再为上层范围(surrounding scope,注:包围eval代码块的范围)引入新变量. 在正常模式下,  代码 eval("var x;") 会给上层函数(surrounding function)或者全局引入一个新的变量 x . 这意味着, 一般情况下,  在一个包含 eval 调用的函数内所有没有引用到参数或者局部变量的名称都必须在运行时才能被映射到特定的定义 (因为 eval 可能引入的新变量会覆盖它的外层变量). 在严格模式下 eval 仅仅为被运行的代码创建变量, 所以 eval 不会使得名称映射到外部变量或者其他局部变量:
```javascript
var x = 17;
var evalX = eval("'use strict'; var x = 42; x");
console.assert(x === 17);
console.assert(evalX === 42);
```
相应的，如果函数eval被在严格模式下的eval(...)以表达的形式调用时，其代码会被当做严格模式下的代码执行，当然也可以在代码中显式开启严格模式，但这样做并不是必须的。
```javascript
function strict1(str){
	"use strict";
	return eval(str); //str中的代码在严格模式下运行
}

function strict2(f, str){
	"use strict";
	return f(str);// 没有直接调用eval(...): 当且仅当str中的代码开启了严格模式时
                 // 才会在严格模式下运行
}

function nonstrict(str){
  return eval(str); // 当且仅当str中的代码开启了"use strict"，str中的代码才会在严格模式下运行
}

strict1("'Strict mode code!'");
strict1("'use strict'; 'Strict mode code!'");
strict2(eval, "'Non-strict code.'");
strict2(eval, "'use strict'; 'Strict mode code!'");
nonstrict("'Non-strict code.'");
nonstrict("'use strict'; 'Strict mode code!'");
```
因此，在 eval 执行的严格模式代码下，变量的行为与严格模式下非 eval 执行的代码中的变量相同。

第三, 严格模式禁止删除声明变量。delete name 在严格模式下会引起语法错误：

```javascript
"use strict";

var x;
delete x; // !!! 语法错误

eval("var y; delete y;"); // !!! 语法错误

```

### 让eval和arguments变的简单
严格模式让arguments和eval少了一些奇怪的行为。两者在通常的代码中都包含了很多奇怪的行为： eval会添加删除绑定，改变绑定好的值，还会通过用它索引过的属性给形参取别名的方式修改形参. 虽然在未来的ECMAScript版本解决这个问题之前，是不会有补丁来完全修复这个问题，但严格模式下将eval和arguments作为关键字对于此问题的解决是很有帮助的。

首先, 名称 eval 和 arguments 不能通过程序语法被绑定(be bound)或赋值. 以下的所有尝试将引起语法错误:
```javascript
"use strict";
eval = 17;
arguments++;
++eval;
var obj = { set p(arguments) { } };
var eval;
try { } catch (arguments) { }
function x(eval) { }
function arguments() { }
var y = function eval() { };
var f = new Function("arguments", "'use strict'; return 17;");
```
第二，严格模式下，参数的值不会随 arguments 对象的值的改变而变化。在正常模式下，对于第一个参数是 arg 的函数，对 arg 赋值时会同时赋值给 arguments[0]，反之亦然（除非没有参数，或者 arguments[0] 被删除）。严格模式下，函数的 arguments 对象会保存函数被调用时的原始参数。arguments[i] 的值不会随与之相应的参数的值的改变而变化，同名参数的值也不会随与之相应的 arguments[i] 的值的改变而变化。

```javascript
"use strict";
var f = function() { return arguments.callee; };
f(); // 抛出类型错误
```
### "安全的" JavaScript
严格模式下更容易写出“安全”的JavaScript。现在有些网站提供了方式给用户编写能够被网站其他用户执行的JavaScript代码。在浏览器环境下，JavaScript能够获取用户的隐私信息，因此这类Javascript必须在运行前部分被转换成需要申请访问禁用功能的权限。没有很多的执行时检查的情况，Javascript的灵活性让它无法有效率地做这件事。一些语言中的函数普遍出现，以至于执行时检查他们会引起严重的性能损耗。做一些在严格模式下发生的小改动，要求用户提交的JavaScript开启严格模式并且用特定的方式调用，就会大大减少在执行时进行检查的必要。

第一，在严格模式下通过`this`传递给一个函数的值不会被强制转换为一个对象。对一个普通的函数来说，`this`总会是一个对象：不管调用时`this`	它本来就是一个对象；还是用布尔值，字符串或者数字调用函数时函数里面被封装成对象的`this`；还是使用`undefined`或者`null`调用函数式`this`代表的全局对象（使用`call`, `apply`或者`bind`方法来指定一个确定的`this`）。这种自动转化为对象的过程不仅是一种性能上的损耗，同时在浏览器中暴露出全局对象也会成为安全隐患，因为全局对象提供了访问那些所谓安全的JavaScript环境必须限制的功能的途径。**所以对于一个开启严格模式的函数，指定的`this`不再被封装为对象，而且如果没有指定`this`的话它值是`undefined`**：
```javascript
"use strict";
function fun() { return this; }
console.assert(fun() === undefined);
console.assert(fun.call(2) === 2);
console.assert(fun.apply(null) === null);
console.assert(fun.call(undefined) === undefined);
console.assert(fun.bind(true)() === true);
```
第二，在严格模式中再也不能通过广泛实现的ECMAScript扩展“游走于”JavaScript的栈中。在普通模式下用这些扩展的话，当一个叫fun的函数正在被调用的时候，fun.caller是最后一个调用fun的函数，而且fun.arguments包含调用fun时用的形参。这两个扩展接口对于“安全”JavaScript而言都是有问题的，因为他们允许“安全的”代码访问"专有"函数和他们的（通常是没有经过保护的）形参。如果fun在严格模式下，那么fun.caller和fun.arguments都是不可删除的属性而且在存值、取值时都会报错：

```javascript
function restricted()
{
  "use strict";
  restricted.caller;    // 抛出类型错误
  restricted.arguments; // 抛出类型错误
}
function privilegedInvoker()
{
  return restricted();
}
privilegedInvoker();
```
第三，严格模式下的arguments不会再提供访问与调用这个函数相关的变量的途径。在一些旧时的ECMAScript实现中arguments.caller曾经是一个对象，里面存储的属性指向那个函数的变量。这是一个安全隐患，因为它通过函数抽象打破了本来被隐藏起来的保留值；它同时也是引起大量优化工作的原因。出于这些原因，现在的浏览器没有实现它。但是因为它这种历史遗留的功能，arguments.caller在严格模式下同样是一个不可被删除的属性，在赋值或者取值时会报错：

```javascript
"use strict";
function fun(a, b)
{
  "use strict";
  var v = 12;
  return arguments.caller; // 抛出类型错误
}
fun(1, 2); // 不会暴露v（或者a，或者b）
```
### 为未来的ECMAScript版本铺平道路

未来版本的ECMAScript很有可能会引入新语法，ECMAScript5中的严格模式就提早设置了一些限制来减轻之后版本改变产生的影响。如果提早使用了严格模式中的保护机制，那么做出改变就会变得更容易。

首先，在严格模式中一部分字符变成了保留的关键字。这些字符包括implements, interface, let, package, private, protected, public, static和yield。在严格模式下，你不能再用这些名字作为变量名或者形参名。
```javascript
function package(protected){ // !!!
  "use strict";
  var implements; // !!!

  interface: // !!!
  while (true)
  {
    break interface; // !!!
  }

  function private() { } // !!!
}
function fun(static) { 'use strict'; } // !!!
```
两个针对Mozilla开发的警告：第一，如果你的JavaScript版本在1.7及以上（你的chrome代码或者你正确使用了`<script type="">`）并且开启了严格模式的话，因为`let`和`yield`是最先引入的关键字，所以它们会起作用。但是网络上用`<script src="">`或者`<script>`...`</script>` 加载的代码，`let`或者`yield`都不会作为关键字起作用；第二，尽管ES5无条件的保留了`class`, `enum`, `expor`, `extends`, `import`和`super`关键字，在Firefox 5之前，Mozilla仅仅在严格模式中保留了它们。

其次，严格模式禁止了不在脚本或者函数层面上的函数声明。在浏览器的普通代码中，在“所有地方”的函数声明都是合法的。这并不在ES5规范中（甚至是ES3）！这是一种针对不同浏览器中不同语义的一种延伸。未来的ECMAScript版本很有希望制定一个新的，针对不在脚本或者函数层面进行函数声明的语法。在严格模式下禁止这样的函数声明对于将来ECMAScript版本的推出扫清了障碍：
```javascript
"use strict";
if (true){
  function f() { } // !!! 语法错误
  f();
}

for (var i = 0; i < 5; i++){
  function f2() { } // !!! 语法错误
  f2();
}

function baz() { // 合法
  function eit() { } // 同样合法
}
```
这种禁止放到严格模式中并不是很合适，因为这样的函数声明方式从ES5中延伸出来的。但这是ECMAScript委员会推荐的做法，浏览器就实现了这一点。

## 浏览器的严格模式
主流浏览器现在实现了严格模式。但是不要盲目的依赖它，因为市场上仍然有大量的浏览器版本只部分支持严格模式或者根本就不支持（比如IE10之前的版本）。严格模式改变了语义。依赖这些改变可能会导致没有实现严格模式的浏览器中出现问题或者错误。谨慎地使用严格模式，通过检测相关代码的功能保证严格模式不出问题。最后，记得在支持或者不支持严格模式的浏览器中测试你的代码。如果你只在不支持严格模式的浏览器中测试，那么在支持的浏览器中就很有可能出问题，反之亦然。

