title: 使用navigator.userAgent简单判断浏览器
date: 2015-08-24 09:41:56
categories: js
tags: js
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/browser-agaent.png
---
#### 简单使用navigator.userAgent判断浏览器

<!-- more -->
![](http://i3.tietuku.com/6d8e1ba88486c179.png)

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>navigator</title>
<script type="text/javascript">
  function validB(){ 
  	var u_agent = navigator.userAgent;
  	var b_name = "不是想用的主流浏览器! "
  	if (u_agent.indexOf("Firefox")>-1) {
  		b_name="Firefox";
  	}else if (u_agent.indexOf("Chrome")>-1) {
  		b_name="Chrome";
  	}else if (u_agent.indexOf("MSIE")>-1 && u_agent.indexOf("Trident")>-1) {
  		b_name="IE(8-10)";
  	};
  	document.write("浏览器："+b_name +'<br>');
  	document.write("u_agent:" +u_agent);
  } 
</script>
</head>
<body>
  <form>
     <input type="button" value="查看浏览器" onclick="validB()"  >
  </form>
</body>
</html>
```