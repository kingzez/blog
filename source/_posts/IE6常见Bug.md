title: IE6常见Bug
date: 2014-08-30 19:39:39
categories: IE
tags: IE6
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/ie-bug.png
---
1、IE6怪异解析之padding与border算入宽高
原因：未加文档声明造成非盒模型解析
解决方法：加入文档声明
<!-- more -->
2、IE6在块元素、左右浮动、设定
marin时造成margin双倍（双边距）
解决方法：`display:inline`

3、以下三种其实是同一种bug，其实也不算是个bug，举个例子：父标签高度20，子标签11，垂直居中，20-11=9，9要分给文字的上面与下面，怎么分？IE6就会与其它的不同，所以，尽量避免。

1）字体大小为奇数之边框高度少1px
解决方法：字体大小设置为偶数或`line-height`为偶数 
2）`line-height`，文本垂直居中差1px
解决方法：`padding-top`代替`line-height`居中，或`line-height`加1或减1
3）与父标签的宽度的奇偶不同的居中造成1px的偏离
解决方法：如果父标签是奇数宽度，则子标签也用奇数宽度;如果是父标签偶数宽度，则子标签也用偶数宽度

4、内部盒模型超出父级时，父级被撑大
解决方法：父标签使用`overflow:hidden;`

5、`line-height`默认行高`bug`
解决方法：`line-height`设值

6、行标签之间会有一小段空白
解决方法：`float`或结构并排(可读性差，不建议)

7、标签高度无法小于19px
解决方法：`overflow: hidden;`

8、左浮元素`margin-bottom`失效
解决方法：显示设置高度 or 父标签设置_padding-bottom代替子标签的margin-bottom or 再放个标签让父标签浮动，子标签
`margin- bottom，`即(`margin-bottom`与`float`不同时作用于一个标签)

9、img于块元素中，底边多出空白

解决方法：父级设置`overflow: hidden;` 或` img { display: block; } `或`_margin: -5px;`

10、li之间会有间距
解决方法：`float: left`;

11、块元素中有文字及右浮动的行元素，行元素换行
解决方法：将行元素置于块元素内的文字前

12、`position`下的`left，bottom`错位
解决方法：为父级(relative层)设置宽高或添加*zoom:1;

13、子级中有设置`position`，则父级`overflow`失效
解决方法：为父级设置`position:relative`;
