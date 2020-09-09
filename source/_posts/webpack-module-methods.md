---
title: webpack 模块方法 【参与开源】
date: 2020-06-23 22:22:25
tags:
---
> 参与 webpack 中文的翻译 https://webpack.docschina.org/api/module-methods/

本章节涵盖了使用 webpack 编译代码的所有方法。在 webpack 打包应用程序时，你可以选择各种模块语法风格，包括 [ES6](https://en.wikipedia.org/wiki/ECMAScript#6th_Edition_-_ECMAScript_2015)，[CommonJS](https://en.wikipedia.org/wiki/CommonJS) 和 [AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition)。


> 虽然 webpack 支持多种模块语法，但我们建议尽量遵循一致的语法，以此避免一些奇怪的行为和 bug。这是一个混合使用了 ES6 和 CommonJS 的[示例](https://github.com/webpack/webpack.js.org/issues/552)，但肯定还会有其他的 bug 产生。


## ES6 (推荐)


webpack 2 支持原生的 ES6 模块语法，意味着你无须额外引入 babel 这样的工具，就可以使用 `import` 和 `export`。但是注意，如果使用其他的 ES6+ 特性，仍然需要引入 babel。webpack 支持以下的方法：


### import


通过 `import` 以静态的方式导入另一个通过 `export` 导出的模块。


```javascript
import MyModule from './my-module.js';
import { NamedExport } from './other-module.js';
```


> 这里的关键词是**静态的**。标准的 `import` 语句中，模块语句中不能以「具有逻辑或含有变量」的动态方式去引入其他模块。关于 import 的更多信息和 `import()` 动态用法，请查看这里的[说明](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)。


### export


`默认`导出整个模块或具名导出模块。


```javascript
// 具名导出
export var Count = 5;
export function Multiply(a, b) {
  return a * b;
}

// 默认导出
export default {
  // Some data...
};
```


### import()


`function(string path):Promise`


动态的加载模块。调用 `import` 的之处，被视为分割点，意思是，被请求的模块和它引用的所有子模块，会分割到一个单独的 chunk 中。


> [ES2015 Loader 规范](https://whatwg.github.io/loader/) 定义了 `import()` 方法，可以在运行时动态地加载 ES2015 模块。


```javascript
if ( module.hot ) {
  import('lodash').then(_ => {
    // Do something with lodash (a.k.a '_')...
  });
}
```


> import() 特性依赖于内置的 [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。如果想在低版本浏览器中使用 `import()`，记得使用像 [es6-promise](https://github.com/stefanpenner/es6-promise) 或者 [promise-polyfill](https://github.com/taylorhakes/promise-polyfill) 这样 polyfill 库，来预先填充(shim) `Promise` 环境。


### import() 中的表达式


不能使用完全动态的 import 语句，例如 `import(foo)`。是因为 `foo` 可能是系统或项目中任何文件的任何路径。


`import()` 必须至少包含一些关于模块的路径信息。打包可以限定于一个特定的目录或文件集，以便于在使用动态表达式时 - 包括可能在 `import()` 调用中请求的每个模块。例如， `import(`./locale/${language}.json`)` 会把 `.locale` 目录中的每个 `.json` 文件打包到新的 chunk 中。在运行时，计算完变量 `language` 后，就可以使用像 `english.json` 或 `german.json` 的任何文件。


```javascript
// 想象我们有一个从 cookies 或其他存储中获取语言的方法
const language = detectVisitorLanguage();
import(`./locale/${language}.json`).then(module => {
  // do something with the translations
});
```


> 使用 [`webpackInclude` and `webpackExclude`](/api/module-methods/#magic-comments) 选项可让你添加正则表达式模式，以减少 webpack 打包导入的文件数量。


#### Magic Comments


内联注释使这一特性得以实现。通过在 import 中添加注释，我们可以进行诸如给 chunk 命名或选择不同模式的操作。


```javascript
// 单个目标
import(
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  /* webpackExports: ["default", "named"] */
  'module'
);

// 多个可能的目标
import(
  /* webpackInclude: /\.json$/ */
  /* webpackExclude: /\.noimport\.json$/ */
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  /* webpackPrefetch: true */
  /* webpackPreload: true */
  `./locale/${language}`
);
```


```javascript
import(/* webpackIgnore: true */ 'ignored-module.js');
```


`webpackIgnore`：设置为 true 时，禁用动态导入解析。


> 注意：将 `webpackIgnore` 设置为 `true` 则不进行代码分割。


`webpackChunkName`: 新 chunk 的名称。 从 webpack 2.6.0 开始，占位符 `[index]` 和 `[request]` 分别支持递增的数字或实际的解析文件名。 添加此注释后，将单独的给我们的 chunk 命名为 [my-chunk-name].js 而不是 [id].js。


`webpackMode`：从 webpack 2.6.0 开始，可以指定以不同的模式解析动态导入。支持以下选项：


- `'lazy'` (默认值)：为每个 `import()` 导入的模块生成一个可延迟加载（lazy-loadable）的 chunk。
- `'lazy-once'`：生成一个可以满足所有 `import()` 调用的单个可延迟加载（lazy-loadable）的 chunk。此 chunk 将在第一次 `import()` 时调用时获取，随后的 `import()` 则使用相同的网络响应。注意，这种模式仅在部分动态语句中有意义，例如 `import(`./locales/${language}.json`)`，其中可能含有多个被请求的模块路径。
- `'eager'`：不会生成额外的 chunk。所有的模块都被当前的 chunk 引入，并且没有额外的网络请求。但是仍会返回一个 resolved 状态的 `Promise`。与静态导入相比，在调用 `import()` 完成之前，该模块不会被执行。
- `'weak'`：尝试加载模块，如果该模块函数已经以其他方式加载，（即另一个 chunk 导入过此模块，或包含模块的脚本被加载）。仍会返回 `Promise`， 但是只有在客户端上已经有该 chunk 时才会成功解析。如果该模块不可用，则返回 rejected 状态的 `Promise`，且网络请求永远都不会执行。当需要的 chunks 始终在（嵌入在页面中的）初始请求中手动提供，而不是在应用程序导航在最初没有提供的模块导入的情况下触发，这对于通用渲染（SSR）是非常有用的。



`webpackPrefetch`：告诉浏览器将来可能需要该资源来进行某些导航跳转。查看指南，了解有关更多信息 [how webpackPrefetch works](/guides/code-splitting/#prefetchingpreloading-modules)。


`webpackPreload`：告诉浏览器在当前导航期间可能需要该资源。 查阅指南，了解有关的更多信息 [how webpackPreload works](/guides/code-splitting/#prefetchingpreloading-modules)。


> 注意：所有选项都可以像这样组合 `/* webpackMode: "lazy-once", webpackChunkName: "all-i18n-data" */`。这会按没有花括号的 JSON5 对象去解析。它会被包裹在 JavaScript 对象中，并使用 [node VM](https://nodejs.org/dist/latest-v8.x/docs/api/vm.html) 执行。所以你不需要添加花括号。


`webpackInclude`：在导入解析（import resolution）过程中，用于匹配的正则表达式。只有匹配到的模块**才会被打包**。


`webpackExclude`：在导入解析（import resolution）过程中，用于匹配的正则表达式。所有匹配到的模块**都不会被打包**。


> 注意，`webpackInclude` 和 `webpackExclude` 不会影响到前缀，例如 `./locale`。


`webpackExports`: 告诉 webpack 在使用动态导入时，只打包这个模块使用的导出项。它可以减小 chunk 的大小。从 [webpack 5.0.0-beta.18](https://github.com/webpack/webpack/releases/tag/v5.0.0-beta.18) 起可用。


> 在 webpack 中使用 `System.import` [不符合提案规则](https://github.com/webpack/webpack/issues/2163)，所以在 [2.1.0-beta.28](https://github.com/webpack/webpack/releases/tag/v2.1.0-beta.28) 后被弃用，并且建议使用 `import()`。


## CommonJS


CommonJS 的目标是为浏览器之外的 JavaScript 指定一个生态系统。webpack 支持以下 CommonJS 的方法：


### require


```javascript
require(dependency: String);
```


已同步的方式检索其他模块的导出。编译器（compiler）会确保依赖项在输出 bundle 中可用。


```javascript
var $ = require('jquery');
var myModule = require('my-module');
```


> 以异步的方式使用，可能不会达到预期效果。


### require.resolve


```javascript
require.resolve(dependency: String);
```


已同步的方式获取模块的 ID。编译器（compiler）会确保依赖项在最终输出 bundle 中可用。建议将其视为不透明值，只能与 `require.cache[id]` 或 `__webpack_require__(id)` 配合使用（最好避免这种用法）。


> 模块 ID 的类型可以是 `number` 或 `string`，具体取决于 [`optimization.moduleIds`](/configuration/optimization/#optimizationmoduleids) 配置。


有关更多模块的信息，详见 [`module.id`](/api/module-variables/#moduleid-commonjs)。


### require.cache


多处引用同一模块，最终只会产生一次模块执行和一次导出。所以，会在运行时（runtime）中会保存一份缓存。删除此缓存，则会产生新的模块执行和新的导出。


> 仅在极少数情况下才需要考虑兼容性！


```javascript
var d1 = require('dependency');
require('dependency') === d1;
delete require.cache[require.resolve('dependency')];
require('dependency') !== d1;
```


```javascript
// in file.js
require.cache[module.id] === module;
require('./file.js') === module.exports;
delete require.cache[module.id];
require.cache[module.id] === undefined;
require('./file.js') !== module.exports; // 理论上是不相等的；实际运行中，则会导致堆栈溢出
require.cache[module.id] !== module;
```


### require.ensure


> `require.ensure()` 是 webpack 特有的，已被 `import()` 取代。
```javascript
require.ensure(
  dependencies: String[],
  callback: function(require),
  errorCallback: function(error),
  chunkName: String
)
```


给定 `dependencies` 参数，将其对应的文件拆分到一个单独的 bundle 中，此 bundle 会被异步加载。当使用 CommonJS 模块语法时，这是动态加载依赖项的唯一方法。这意味着，可以在模块执行时才允许代码，只有在满足特定条件时才会加载 `dependencies`。


> 这个特性依赖内置的 [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。如果你在低版本浏览器中使用 `require.ensure`，记得使用像 [es6-promise](https://github.com/stefanpenner/es6-promise) 或 [promise-polyfill](https://github.com/taylorhakes/promise-polyfill) 这样的 polyfill 库，预先填充（shim）`Promise` 环境。


```javascript
var a = require('normal-dep');

if ( module.hot ) {
  require.ensure(['b'], function(require) {
    var c = require('c');

    // Do something special...
  });
}
```


按照上面指定的顺序，`require.ensure` 支持以下参数：


- `dependencies`：字符串数组，声明 `callback` 回调函数中所需要的所有模块。
- `callback`：当依赖项加载完成后，webpack 将会执行此函数，`require` 函数的实现，作为参数传入此函数中。当程序运行需要依赖时，可以使用 `require()` 来加载依赖。函数体可以使用此参数，来进一步执行 `require()` 模块。
- `errorCallback`：当 webpack 加载依赖失败时会执行此函数。
- `chunkName`：由 `require.ensure` 创建的 chunk 的名称。通过将相同 `chunkName` 传递给不同的 `require.ensure` 调用，我们可以将其代码合并到一个单独的 chunk 中，从而只产生一个浏览器必须加载的 bundle。



> 虽然将 `require` 的实现作为参数（可以使用任意名称）传递给 `callback` 函数，例如，`require.ensure([], function(request) { request('someModule'); })` 则不能被 webpack 静态解析器处理，因此还是使用 `require` 作为参数名，例如，`require.ensure([], function(require) { require('someModule') })`。


## AMD


AMD（Asynchronous Module Definition）是一种定义了用于编写和加载模块接口的 JavaScript 规范。


### define (通过 factory 方法导出)
```javascript
define([name: String], [dependencies: String[]], factoryMethod: function(...))
```


如果提供了 `dependencies` 参数，就会调用 `factoryMethod` 方法，并（以相同的顺序）导出每个依赖项。如果未提供 `dependencies` 参数，调用 `factoryMethod` 方法时会传入 `require` , `exports` 和 `module`（用于兼容！）。如果此方法返回一个值，则返回值会作为此模块的导出。由编译器（compiler）来确保依赖项在最终输出的 bundle 中可用。


> 注意：webpack 会忽略 `name` 参数。


```javascript
define(['jquery', 'my-module'], function($, myModule) {
  // 使用 $ 和 myModule 做一些操作...

  // 导出一个函数
  return function doSomething() {
    // ...
  };
});
```


> 上面的写法不能在异步函数中使用。


### define (通过 value 导出)
```javascript
define(value: !Function)
```


这种方式只将提供的 `value` 导出。这里的 `value` 可以是除函数以外的任何值。


```javascript
define({
  answer: 42
});
```


> 上面的写法不能在异步函数中使用。


### require (AMD 版本)
```javascript
require(dependencies: String[], [callback: function(...)])
```


与 `require.ensure` 类似，给定 `dependencies` 参数，将其对应的文件拆分到一个单独的 bundle 中，此 bundle 会被异步加载。然后会调用 `callback` 回调函数，并传入 `dependencies` 数组中的每个项导出。


> 这个特性依赖内置的 [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。如果你在低版本浏览器中使用 `require.ensure`，记得使用像 [es6-promise](https://github.com/stefanpenner/es6-promise) 或 [promise-polyfill](https://github.com/taylorhakes/promise-polyfill) 这样的 polyfill 库，预先填充（shim）`Promise` 环境。


```javascript
require(['b'], function(b) {
  var c = require('c');
});
```


> 这里没有提供命名 bundle 名称的选项。


## 标签模块(Labeled Modules)


webpack 内置的 `LabeledModulesPlugin` 插件，允许你使用下面的方法导出和导入模块：


### export 标签


导出给定的 `value`。标签可以出现在函数声明或变量声明之前。函数名或变量名是导出值的标识符。
```javascript
export: var answer = 42;
export: function method(value) {
  // Do something...
};
```


> 在异步函数中使用，可能不会达到预期的效果。


### require 标签


在当前作用域下，依赖项的所有导出均可用。`require` 标签可以放置在一个字符串之前。依赖模块必须使用 `export` 标签导出值。CommonJS 或 AMD 模块无法通过这种方式使用。


**some-dependency.js**
```javascript
export: var answer = 42;
export: function method(value) {
  // Do something...
};
```
```javascript
require: 'some-dependency';
console.log(answer);
method(...);
```


## Webpack


除了上述模块语法之外，还允许使用一些 webpack 特定的方法：


### require.context

```javascript
require.context(
  directory: String,
  includeSubdirs: Boolean /* 可选的，默认值是 true */,
  filter: RegExp /* 可选的，默认值是 /^\.\/.*$/，所有文件 */,
  mode: String  /* 可选的， 'sync' | 'eager' | 'weak' | 'lazy' | 'lazy-once'，默认值是 'sync' */
)
```


指定一系列依赖项，通过使用 `directory` 的路径，以及 `includeSubdirs` ，`filter` 选项，进行更细粒度的模块引入，使用 `mode` 定义加载方式。以此可以很容易的解析模块：


```javascript
var context = require.context('components', true, /\.html$/);
var componentA = context.resolve('componentA');
```


如果 `mode` 设置为 `lazy`，基础模块将以异步方式加载：


```javascript
var context = require.context('locales', true, /\.json$/, 'lazy');
context('localeA').then(locale => {
  // do something with locale
});
```


`mode` 的可用模式及说明的完整列表在 [`import()`](#import-1) 文档中进行了描述。


### require.include
```javascript
require.include(dependency: String)
```


引入一个不需要执行的 `dependency`，这样可以用于优化输出 chunk 中依赖模块的位置。


```javascript
require.include('a');
require.ensure(['a', 'b'], function(require) { /* ... */ });
require.ensure(['a', 'c'], function(require) { /* ... */ });
```


这会产生以下输出：


- entry chunk: `file.js` and `a`
- anonymous chunk: `b`
- anonymous chunk: `c`



不使用 `require.include('a')`，输出的两个匿名 chunk 都会有模块 a。


### require.resolveWeak


与 `require.resolve` 类似，但是不会把 `module` 引入到 bundle 中。这就是所谓的“弱（weak）”依赖。


```javascript
if(__webpack_modules__[require.resolveWeak('module')]) {
  // 当模块可用时，执行一些操作……
}
if(require.cache[require.resolveWeak('module')]) {
  // 在模块加载完成之前，执行一些操作……
}

// 你可以像执行其他 require/import 方法一样，
// 执行动态解析上下文 resolves ("context")。
const page = 'Foo';
__webpack_modules__[require.resolveWeak(`./page/${page}`)];
```


> `require.resolveWeak` 是_通用渲染_（服务器端渲染 SSR + 代码分割 Code Splitting）的基础。例如在 [react-universal-component](https://github.com/faceyspacey/react-universal-component) 等包中的用法。它允许代码在服务器端和客户端初始页面的加载上同步渲染。它要求手动或以某种方式提供 chunk。它可以在不需要指示应该被打包的情况下引入模块。它与 `import()` 一起使用，当用户导航触发额外的导入时，它会被接管。
