title: js通过hash方式数组去重
date: 2016-05-17 22:53:15
categories: js
tags: js
---
之前面试中遇到过数组去重的问题，方法很多，但是相对来说比较好的方式是通过hash方式来进行去重，这样的效率要高很多，以下是具体代码：
<!-- more -->

```
var newArr = [1,2,2,3,3,3,3,4,4,5,5,6,6,7,8,8,8,9,9,9,9];

function killArray(arr){
	var hash = {},
	    result = [];

	for(var i=0; i<arr.length; i++){
		if(!hash[arr[i]]){
		    hash[arr[i]] = true;
		    result.push(arr[i]);
		}
	}
	return result;
}

killArray(newArr); //输出[1,2,3,4,5,6,7,8,9]
```
完成~