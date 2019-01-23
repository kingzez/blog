title: 初试Mithril.js
date: 2016-11-15 14:00:01
categories: js
tags: mithril.js
---
初试Mithril.js<!-- more -->

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>mithril</title>
    <script src="node_modules/mithril/mithril.min.js"></script>
</head>
<body>
</body>
<script>
	// this simplicity , we use this component to namespace the model classes
	var todo = {};
	// the Todo class has two properties
	todo.Todo = function(data) {
		this.description = m.prop(data.description);
		this.done = m.prop(false);
	};
	// TodoList class is a list of Todo's
	todo.TodoList = Array;
	// the view-model tracks a running list of todos,
	// store a description for new todos before they are created
	// and takes care of the logic surrounding when adding is permitted
	// and clearing the input after adding a todo to the list
	todo.vm = (function() {
		var vm = {};
		vm.init = function() {
			// a running list of todos
			vm.list = new todo.TodoList();
			// a slot to store the name of a new todo before it is created
			vm.description = m.prop("");
			// adds a todo to the list, and clears the description filed for user convenience
			vm.add = function() {
				if (vm.description) {
					vm.list.push(new todo.Todo({description: vm.description()}));
					vm.description("");
				}
			};
		}
		return vm;
	}())
	// the controller defineds what part of the model is relevant for the current page
	// in our case, there's only one view-model that handles everything
	todo.controller = function() {
		todo.vm.init()
	};
	// here is the view
	todo.view = function() {
		return [
			m("input", {onchange: m.withAttr("value", todo.vm.description), value: todo.vm.description()}),
			m("button", {onclick: todo.vm.add}, "Add"),
			m("table", [
				todo.vm.list.map(function(task, index) {
					return m("tr", [
						m("td", [
							m("input[type=checkbox]", {onclick: m.withAttr("checked", task.done), checked: task.done()})
						]),
						m("td", {style: {textDecoration: task.done() ? "line-through" : "none"}}, task.description()),
					])
				})
			])
		]
	};
	// initialize the application
	m.mount(document.body, {controller: todo.controller, view: todo.view});
</script>
</html>
```
