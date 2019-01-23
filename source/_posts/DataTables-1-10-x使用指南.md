title: DataTables 1.10.x使用指南
date: 2017-03-03 12:51:16
categories: js
tags: js datatables
---

DataTables是一个jQuery的表格插件。这是一个高度灵活的工具，依据的基础逐步增强，这将增加先进的互动控制，支持任何HTML表格。主要特点：
<!-- more -->
- 自动分页处理
- 即时表格数据过滤
- 数据排序以及数据类型自动检测
- 自动处理列宽度
- 可通过CSS定制样式
- 支持隐藏列
- 易用
- 可扩展性和灵活性
- 国际化
- 动态创建表格
- 免费的

# 安装
## 依赖
因为DataTables是依赖jquery，所以要先引入jquery，再引入`jquery.dataTables.min.js`。

```javascript
//code.jquery.com/jquery-1.12.4.js
https://cdn.datatables.net/1.10.13/js/jquery.dataTables.min.js
```
## HTML
对于数据表能够增强HTML表格效果，表格必须是有效的，要有`<thead>`,`<tbody>`标签，可选的有`<tfoot>`标签。
```html
<table id="table_id" class="display">
    <thead>
        <tr>
            <th>Column 1</th>
            <th>Column 2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Row 1 Data 1</td>
            <td>Row 1 Data 2</td>
        </tr>
        <tr>
            <td>Row 2 Data 1</td>
            <td>Row 2 Data 2</td>
        </tr>
    </tbody>
</table>
```
## 安装javascript/css文件
共三种不同的方式安装

- 使用DataTables CDN
- 下载后本地引入
- 通过NPM或Bower包管理工具安装

### CDN
```html
<link rel="stylesheet" type="text/css" href="//cdn.datatables.net/1.10.13/css/jquery.dataTables.css">
<script type="text/javascript" charset="utf8" src="//cdn.datatables.net/1.10.13/js/jquery.dataTables.js"></script>
```
### 本地安装
下载安装包
```html
<link rel="stylesheet" type="text/css" href="/DataTables/datatables.css">
<script type="text/javascript" charset="utf8" src="/DataTables/datatables.js"></script>
```
### NPM
```bash
npm install datatables.net    # Core library
npm install datatables.net-dt # Styling
```
## 初始化表格
```javascript
$(document).ready( function () {
    $('#table_id').DataTable();
} );
```

# 数据

- 数据处理模式
- 数据类型
- 数据源

## 处理模式
分为两种：客户端处理模式和服务端处理模式。

客户端处理模式：完整的数据集加载和数据处理都是在浏览器中完成。

服务端处理模式：通过Ajax请求数据绘画表格，服务端只返回数据，数据处理由服务端来完成。

## 数据类型
分为数组（Array）、对象（Object）、实例（new MyClass()）。

- 数组：数组数据
```javascript
var data = [
    [
        "Tiger Nixon",
        "System Architect",
        "Edinburgh",
        "5421",
        "2011/04/25",
        "$3,120"
    ],
    [
        "Garrett Winters",
        "Director",
        "Edinburgh",
        "8422",
        "2011/07/25",
        "$5,300"
    ]
]
```

表格初始化
```javacript
$('#example').DataTable( {
    data: data
} );
```

效果图为

- 对象：对象数据
```javascript
[    {
        "name":       "Tiger Nixon",
        "position":   "System Architect",
        "salary":     "$3,120",
        "start_date": "2011/04/25",
        "office":     "Edinburgh",
        "extn":       "5421"
    },
    {
        "name":       "Garrett Winters",
        "position":   "Director",
        "salary":     "$5,300",
        "start_date": "2011/07/25",
        "office":     "Edinburgh",
        "extn":       "8422"
    }
]
```
表格初始化为
```javascript
$('#example').DataTable( {
    data: data,
    columns: [
        { data: 'name' },
        { data: 'position' },
        { data: 'salary' },
        { data: 'office' }
    ]
} );
```
效果图为

实例化：
```javascript
function Employee ( name, position, salary, office ) {
    this.name = name;
    this.position = position;
    this.salary = salary;
    this._office = office;

    this.office = function () {
        return this._office;
    }
};
```

```javascript
$('#example').DataTable( {
    data: [
        new Employee( "Tiger Nixon", "System Architect", "$3,120", "Edinburgh" ),
        new Employee( "Garrett Winters", "Director", "$5,300", "Edinburgh" )
    ],
    columns: [
        { data: 'name' },
        { data: 'salary' },
        { data: 'office' },
        { data: 'position' }
    ]
} );
```
效果图如下

## 数据源

- DOM (html文件)
- Javascript
- Ajax 数据源

### Ajax
加载数据
```javascript
$('#myTable').DataTable( {
    ajax: '/api/myData'
} );
```

可以使用json数据源，数组数据源，对象数据源。以下举例说明：

1.简单的数组数据
```javascript
[    {
        "name": "Tiger Nixon",
        "position": "System Architect",
        "salary": "$320,800",
        "start_date": "2011/04/25",
        "office": "Edinburgh",
        "extn": "5421"
    },
    ...
}
```

使用方法如下
```javascript
$('#myTable').DataTable( {
	ajax: {
        url: '/api/myData',
        dataSrc: ''
    },
    columns: [ ... ]
} );
```

2.对象数据
```javascript
{
    "data": [
        {
            "name": "Tiger Nixon",
            "position": "System Architect",
            "salary": "$320,800",
            "start_date": "2011/04/25",
            "office": "Edinburgh",
            "extn": "5421"
        },
        ...
    ]
}
```

使用方法如下
```javascript
$('#myTable').DataTable( {
	ajax: '/api/myData',
    columns: [ ... ]
} );

// or!

$('#myTable').DataTable( {
    ajax: {
        url: '/api/myData',
        dataSrc: 'data'
    },
    columns: [ ... ]
} );
```
3.对象数据对应关系
 ```javascript
{  "name": "Tiger Nixon",
    "position": "System Architect",
    "salary": "$320,800",
    "start_date": "2011/04/25",
    "office": "Edinburgh",
    "extn": "5421"
}
 $('#myTable').DataTable( {
 	ajax: ...,
    columns: [
        { data: 'name' },
        { data: 'position' },
        { data: 'salary' },
        { data: 'state_date' },
        { data: 'office' },
        { data: 'extn' }
    ]
} );
```

4.嵌套对象关系
```javascript
{
    "name": "Tiger Nixon",
    "hr": {
        "position": "System Architect",
        "salary": "$320,800",
        "start_date": "2011/04/25"
    },
    "contact": {
        "office": "Edinburgh",
        "extn": "5421"
    }
}
$('#myTable').DataTable( {
	ajax: ...,
    columns: [
        { data: 'name' },
        { data: 'hr.position' },
        { data: 'hr.salary' },
        { data: 'hr.state_date' },
        { data: 'contact.office' },
        { data: 'contact.extn' }
    ]
} );
```
# [在线例子](https://www.datatables.net/examples/ajax/)

# 配置项
使用案例说明配置项，以下是常用的配置项，详细请到官网[https://www.datatables.net/](https://www.datatables.net/)查询。
```javacript
var reportMonthtable = $('#monthReport').DataTable({
    retrieve: true, //防止多次初始化
    data: dataSet, //数据集
    columns: [
        { "data": "ruleName" },
        { "data": "mainWhere" },
        { "data": "UV" },
        { "data": "PV" },
        { "data": "sessions" }
    ], //表头名称
    paging: true, //显示翻页
    lengthChange: false, //关闭每页显示
    "info": false, //关闭当前显示表格信息
    searching: false, //关闭搜索框
    scrollY: 200, //自定义滚动高度
    iDisplayLength: jtPageSize, //每页显示条数
    order: [], //插件bug 取代默认值
    columnDefs: [{
            orderable: false, //禁用排序
            targets: [0, 1] //指定的列
        },
        // 将mainWhere列变为蓝色
        {
            targets: [1], // 目标列位置，下标从0开始
            data: "mainWhere", // 数据列名
            render: function(data, type, full) { // 返回自定义内容
                return "<span style='color:#337ab7;'>" + data + "</span>";
            }
        }
    ],
    bProcessing: true, //加载进度条
    stateSave: true, //保存翻页状态
    //语言配置
    language: {
        sProcessing: "正在加载中......",
        sLengthMenu: "每页显示 _MENU_ 条记录",
        sZeroRecords: "该时间段无统计数据",
        sEmptyTable: "该时间段无统计数据",
        sInfoEmpty: "",
        sInfo: "当前显示 _START_ 到 _END_ 条，共 _TOTAL_ 条记录",
        sInfoFiltered: "数据表中共为 _MAX_ 条记录",
        sSearch: "",
        oPaginate: {
            sFirst: "首页",
            sPrevious: "上一页",
            sNext: "下一页",
            sLast: "末页"
        }
    }
});
```

# 注意点
结合项目中与angularjs结合使用，angular通过service获取后端数据，将数据集导入初始化中，但是该插件暂时不支持`promise`，只能通过变量数据集，这将导致数据变化后表格的数据不能及时更新，需要刷新页面才可以，这不是我们想要的效果，可以通过ui-router中的`$state`服务的`$state.reload`来实现reload操作，无需刷新页面即可达到数据变化后页面能够更新重绘。

# 参考文章：
[https://www.datatables.net/](https://www.datatables.net/)

