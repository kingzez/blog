title: angularjs与MDL的整合
date: 2016-09-16 11:03:25
categories: js
tags: [angularjs, MDL]
---
很喜欢Material Design的风格，最近做项目打算用[Material Design Lite](https://getmdl.io)<!-- more -->搭配angularjs 1.3.x,首先需要注意的是MDL在动态网页中需要更新注册组件即`upgradeAllRegistered`,起初不了解，在angular中引用MDL的js和css，但是按钮的涟漪效果以及其他涉及js操作的样式都无效，（一脸懵逼中……）后来仔细研读官方文档后才注意到这个小坑，动态网页中需要随时更新注册组件才能生效，官方文档是这么说的：
>Material Design Lite will automatically register and render all elements marked with MDL classes upon page load. However in the case where you are creating DOM elements dynamically you need to register new elements using the `upgradeElement` function. Here is how you can dynamically create the same raised button with ripples shown in the section above:

```
<div id="container"/>
<script>
  var button = document.createElement('button');
  var textNode = document.createTextNode('Click Me!');
  button.appendChild(textNode);
  button.className = 'mdl-button mdl-js-button mdl-js-ripple-effect';
  componentHandler.upgradeElement(button);
  document.getElementById('container').appendChild(button);
</script>
```
找到原因后就要解决问题了，在angularjs中如何更新组件呢？因为这个更新是每时每刻都在运行的，所以要加在angular的启动文件app.js中，需要注意的是run加的位置，区分config和run的区别，config阶段是给了ng上下文一个针对constant与provider修改其内部属性的一个阶段，而run阶段是在config之后的在运行独立的代码块，通常写法runBlock，简单的说一下就是ng启动阶段是 config-->run-->compile/link，那么解决方法是：
```
angular.module('app',[])
	.run(['$rootScope', '$timeout', function($rootScope, $timeout) {
		$rootScope.$on('$viewContentLoaded', () => {
			$timeout(() => {
			componentHandler.upgradeAllRegistered();
			})
		})
	}]);
```
那么开始后续的开发~