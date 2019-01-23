title: $scope与controller as区别
date: 2016-09-24 23:54:56
categories: js
tags: angularjs
---
AngularJS中controller的实现主要有两种方法：
<!-- more -->
- 第一种为：我们熟悉的将model绑定到$scope上，在html页面中`ng-controller="SettingsController"`。
- 第二种为：将方法和属性直接绑定到controller上，在html页面中`ng-controller="SettingsController as setting"`。
第一种我们都很熟悉，那么这两种到底有什么区别呢？例子来自angularJS官网。

*controller as*
index.html
```
<div id="ctrl-as-exmpl" ng-controller="SettingsController1 as settings">
  Name: <input type="text" ng-model="settings.name"/>
  [ <a href="" ng-click="settings.greet()">greet</a> ]<br/>
  Contact:
  <ul>
    <li ng-repeat="contact in settings.contacts">
      <select ng-model="contact.type">
         <option>phone</option>
         <option>email</option>
      </select>
      <input type="text" ng-model="contact.value"/>
      [ <a href="" ng-click="settings.clearContact(contact)">clear</a>
      | <a href="" ng-click="settings.removeContact(contact)">X</a> ]
    </li>
    <li>[ <a href="" ng-click="settings.addContact()">add</a> ]</li>
 </ul>
</div>

```

app.js

```
angular.module('controllerAsExample', [])
  .controller('SettingsController1', SettingsController1);

function SettingsController1() {
  this.name = "John Smith";
  this.contacts = [
    {type: 'phone', value: '408 555 1212'},
    {type: 'email', value: 'john.smith@example.org'} ];
}

SettingsController1.prototype.greet = function() {
  alert(this.name);
};

SettingsController1.prototype.addContact = function() {
  this.contacts.push({type: 'email', value: 'yourname@example.org'});
};

SettingsController1.prototype.removeContact = function(contactToRemove) {
 var index = this.contacts.indexOf(contactToRemove);
  this.contacts.splice(index, 1);
};

SettingsController1.prototype.clearContact = function(contact) {
  contact.type = 'phone';
  contact.value = '';
};
```
*$scope*
index.html
```
<div id="ctrl-exmpl" ng-controller="SettingsController2">
  Name: <input type="text" ng-model="name"/>
  [ <a href="" ng-click="greet()">greet</a> ]<br/>
  Contact:
  <ul>
    <li ng-repeat="contact in contacts">
      <select ng-model="contact.type">
         <option>phone</option>
         <option>email</option>
      </select>
      <input type="text" ng-model="contact.value"/>
      [ <a href="" ng-click="clearContact(contact)">clear</a>
      | <a href="" ng-click="removeContact(contact)">X</a> ]
    </li>
    <li>[ <a href="" ng-click="addContact()">add</a> ]</li>
 </ul>
</div>
```
app.js
```
angular.module('controllerExample', [])
  .controller('SettingsController2', ['$scope', SettingsController2]);

function SettingsController2($scope) {
  $scope.name = "John Smith";
  $scope.contacts = [
    {type:'phone', value:'408 555 1212'},
    {type:'email', value:'john.smith@example.org'} ];

  $scope.greet = function() {
    alert($scope.name);
  };

  $scope.addContact = function() {
    $scope.contacts.push({type:'email', value:'yourname@example.org'});
  };

  $scope.removeContact = function(contactToRemove) {
    var index = $scope.contacts.indexOf(contactToRemove);
    $scope.contacts.splice(index, 1);
  };

  $scope.clearContact = function(contact) {
    contact.type = 'phone';
    contact.value = '';
  };
}
```

定义controller时不用显式的依赖$scope，好处就是一个普通的函数定义，例子中的SettingsController2就是所谓的POJO（Plain Old Javascript Object，Java里的概念），这样的Object与框架无关，里面只有逻辑。即便有一天你的项目不再使用AngularJS了，依然可以很方便的重用和移植这些逻辑。另外，从测试的角度看，这样的Object也是单元测试友好的。单元测试强调的就是孤立其他依赖元素，而POJO恰恰满足这个条件，可以单纯的去测试这个函数的输入输出，而不用费劲的去模拟一个假的$scope。

另外，还有一个比较牵强的好处：防止滥用$scope的$watch，$on，$broadcast方法。可能刚刚就有人想问了，不依赖$scope我怎么watch一个model，怎样广播和响应事件。答案是没法弄，这些事还真是只有$scope能干。但很多时候在controller里watch一个model是很多余的，这样做会明显的降低性能。所以，当你本来就依赖$scope的时候，你会习惯性的调用这些方法来实现自己的逻辑。但当使用controller as的时候，由于没有直接依赖$scope，使用watch前你会稍加斟酌，没准就思考到了别的实现方式了呢。