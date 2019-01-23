title: MacOS自带Apache的坑
date: 2016-09-22 11:16:02
categories: Mac
tags: apache
---
刚刚升级了10.12，`sudo apachectl start`启动apache的时候瞬间懵逼了，
<!--more-->
``` bash
sudo apachectl start
(48)Address already in use: AH00072: make_sock: could not bind to address [::]:80
(48)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:80
no listening sockets available, shutting down
AH00015: Unable to open logs
```

明明没有启动，也没做什么，后来发现原因是我使用的不是mac自带的apache，升级之后就悲剧，因为在新的系统中，apache是默认启动的，且不支持删除，所以解决方法是
- 停止apache
```
sudo apachectl -k stop
```
- 关闭apache随系统启动
```
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
这样就可以了，再次`sudo apachectl start` 就是自己安装的apache服务了。