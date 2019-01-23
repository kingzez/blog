title: github的静态博客用万网解析
date: 2015-07-14 00:15:21
categories: git
tags: [git,github]
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/github-pages.png
---
网上关于github的解析都是一带而过，下面详细说一下步骤   
<!-- more -->
1.首先添加解析记录类型A，主机类型默认不填，记录值填写username.github.io所ping
的值。  
2.添加解析记录类型CNAME，主机类型为WWW，记录值填写你的username.github.io   
3.最后在你的静态博客根目录下创建CNAME文件，文件内容填写你所购买的域名。   
解析完成。
![](http://i1.tietuku.com/0b9e9b3a72812b5b.png)   

![](http://i3.tietuku.com/9096e368c62a60c8.png)