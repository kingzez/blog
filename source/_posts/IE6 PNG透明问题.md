title: IE6 PNG透明问题
date: 2014-08-22 16:38:41
categories: IE
tags: IE6
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/ie-png.png
---
对于IE6 png 透明，解决方法很多，本文介绍的是博主经常用的的两种方法：
<!-- more -->
####第一种是：Alpha（阿尔法）修复法
复制并粘贴iepngfix.htc和blank.gif到您的网站的文件夹。

CSS选择器，必须包括标签/元素对你想要PNG支持——基本上，给它一个逗号分隔的列表标记的使用。它还必须包括正确的路径。HTC相对于HTML文档位置（不相关的CSS文件！）。例如，可能像这样：
```
<style type="text/css">
img, div, a, input { behavior: url(/css/resources/iepngfix.htc) }
</style>
```
使用起来非常简单，就这样，就可以达到IE6下png透明
[iepngfix.htc blank.gif下载包](http://www.twinhelix.com/css/iepngfix/iepngfix.zip)
####第二种是：Unit PNG Fix 
* 1.非常小的javascript文件:1kb!
* 2.解决因为IE的滤镜属性所带来的影响.
* 3.无论是img元素或background-image属性,都能有效果.
* 4.在加载页面之前就能自动运行.或者就一丁点的元素.
* 5.允许自动高宽.
* 6.使用起来超级简单.
如何使用:
1).下载zip 然后,添加下面的代码到你页面的头部.(一定要确保路径的正确)
```
 <!--[if lt IE 7]>
        <script type="text/javascript" src="unitpngfix.js"></script>
<![endif]--> 
```
2).添加clear.gif到你的images 文件夹中.在js文件中,修改"var clear="images/clear.gif" 路径,为你存放clear.gif的文件路径. 

3). 你的整个项目的png图片都实现了透明效果.的确非常简单吧?就2个步骤,就实现了整个站点所有png的透明效果.
[Uint PNG Fix下载包](http://labs.unitinteractive.com/downloads/unitpngfix.zip)	