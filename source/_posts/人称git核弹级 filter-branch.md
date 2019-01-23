title: '人称git核弹级 filter-branch '
date: 2015-12-20 21:05:59
categories: git
tags: [git,github]
---

都说“占小便宜吃大亏”,一点不假，听说GitHub Education的服务很不错，就用自己的edu.cn邮箱把GitHub的邮箱改了，接着去Get the Student Developer Pack。

<!-- more -->

静待回馈email，But！！！被拒绝了，原因是不认edu.cn的邮箱，好吧算了被拒绝也没什么好说的了，回到我的GitHub主页，顿时眼前一白，我的contributions图消失了，虽说不是很多但也是代表自己的contributions，好了不说废话了，解决竟是git的核弹级的 filter-branch。解决方法为全局性地更换电子邮件地址代码如下

```
git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = w@W-MacBook-Pro.local ];
        then
                GIT_AUTHOR_NAME=kingzez;
                GIT_AUTHOR_EMAIL=kingzez@sina.com;
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD
```
接下来解释一下代码含义：这次的一个失误让我也发现了一个容易忽视的问题，在重装Mac后忘记了配置本地的git config，所以造成了我本地不合法的git用户邮箱（w@W-MacBook-Pro.local）其中上传的部分内容不在contributions上显示，（另外还有我的edu.cn的邮箱，针对我个人而言需要修改两次，）原因是contributions图是对应git用户配置显示的，这是我更改了邮箱后图消失的原因，Github系统找不到现在邮箱的git用户，所以我现在要操作的是

```
git config --global user.name "kingzez"
```
```
git config --global user.email "kingzez@sina.com"
```
操作后为了安全起见，在检查一遍
```
git config --list
```
然后操作核弹级的`filter-branch --commit-filter` 这个必须要小心，千万要确认自己修改的邮箱是正确的，这个会遍历并重写所有提交历史到你新的邮箱上，命令完成后需要强推 `git push -f`，到Github主页刷新，我的contributions图又回来了，但是其中可能是以前的操作不当恢复的不是很完整，能恢复大部分已经很好了，原本只是改了一个邮箱，结果折腾这么一大圈。

 
