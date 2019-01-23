title: Blog无图片时添加随机thumbnail
date: 2016-05-18 19:03:25
categories: js
tags: js
<!-- photo: 
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/blog-thumbnail.png -->
---
有的文章中没有添加如上这种banner图，所以呢，博客右侧的Recent的缩略图只会显示默认的图，<!-- more -->有时候连续五篇文章都没有图那就连续五张缩略图一样了，很不美观，不喜欢~所以就把它改成没有缩略图情况下随机变换缩略图，不多说，上代码：
首先需要通过Math对象生成出一个随机数，random方法可以生成一个0~1之间的随机数，得到的数再乘10，结果即为0~10之间的一个随机数，再通过round方法对该随机数进行四舍五入得到一个整数
```
var randomBgIndex = Math.round(Math.random() * 10);
```
再声明一个数组用来存放缩略图的路径
```
var thumbnailBg = []; 
	thumbnailBg[0] = "css/images/thumb-default-small.png"; 
	thumbnailBg[1] = "css/images/thumb-default-small1.png"; 
	thumbnailBg[2] = "css/images/thumb-default-small2.png"; 
	thumbnailBg[3] = "css/images/thumb-default-small3.png"; 
	thumbnailBg[4] = "css/images/thumb-default-small4.png"; 
	thumbnailBg[5] = "css/images/thumb-default-small5.png"; 
	thumbnailBg[6] = "css/images/thumb-default-small6.png"; 
	thumbnailBg[7] = "css/images/thumb-default-small7.png"; 
	thumbnailBg[8] = "css/images/thumb-default-small8.png"; 
	thumbnailBg[9] = "css/images/thumb-default-small9.png"; 
	thumbnailBg[10] = "css/images/thumb-default-small10.png"; 
<span style="background-image:url(<%= thumbnailBg[randomBgIndex] %>)"  class="thumbnail-image thumbnail-none"></span>

```
<% %>这里是blog用的ejs语法`thumbnailBg[randomBgIndex]`用来取随机产生的值。

完整的代码：
```
var randomBgIndex = Math.round(Math.random() * 10);
var thumbnailBg = []; 
	thumbnailBg[0] = "css/images/thumb-default-small.png"; 
	thumbnailBg[1] = "css/images/thumb-default-small1.png"; 
	thumbnailBg[2] = "css/images/thumb-default-small2.png"; 
	thumbnailBg[3] = "css/images/thumb-default-small3.png"; 
	thumbnailBg[4] = "css/images/thumb-default-small4.png"; 
	thumbnailBg[5] = "css/images/thumb-default-small5.png"; 
	thumbnailBg[6] = "css/images/thumb-default-small6.png"; 
	thumbnailBg[7] = "css/images/thumb-default-small7.png"; 
	thumbnailBg[8] = "css/images/thumb-default-small8.png"; 
	thumbnailBg[9] = "css/images/thumb-default-small9.png"; 
	thumbnailBg[10] = "css/images/thumb-default-small10.png"; 
<span style="background-image:url(<%= thumbnailBg[randomBgIndex] %>)"  class="thumbnail-image thumbnail-none"></span>
```
OK~