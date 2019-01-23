title: DOM 简单操作实现多选反选
date: 2015-08-24 23:59:11
categories: js
tags: js
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/dom.png
---
#### DOM简单操作实现多选反选
<!-- more -->
```
<!DOCTYPE HTML>
<html>
 <head>
 <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
 <title>DOM</title>
 </head>
    <body>
        <form>
          Choose one:<br>
          <input type="checkbox" name="hobby" id="hobby1">  音乐
          <input type="checkbox" name="hobby" id="hobby2">  登山
          <input type="checkbox" name="hobby" id="hobby3">  游泳
          <input type="checkbox" name="hobby" id="hobby4">  阅读
          <input type="checkbox" name="hobby" id="hobby5">  打球
          <input type="checkbox" name="hobby" id="hobby6">  跑步 <br>
          <input type="button" value = "全选" onclick = "checkall();">
          <input type="button" value = "全不选" onclick = "clearall();">
          <p>input your number，number1 to 6:</p>
          <input id="wb" name="wb" type="text" >
          <input name="ok" type="button" value="确定" onclick = "checkone();">
        </form>
<script type="text/javascript">
function checkall(){
var hobby = document.getElementsByTagName("input");          
for(i=0; i<hobby.length; i++)
if(hobby[i].type =="checkbox"){
    hobby[i].checked = true;
}
}
function clearall(){
var hobby = document.getElementsByName("hobby");
for(i = 0; i < hobby.length; i++){
    hobby[i].checked =false;
}         
}

function checkone(){
var j=document.getElementById("wb").value;
var hobby = document.getElementById("hobby" + j);
hobby.checked = true;
}

</script>
    </body>
</html>
```