title: 如何完全删除gitosis
date: 2015-10-19 23:04:24
categories: git
tags: [git,gitosis]
---
如果你是用apt安装的，那很简单apt-get purge gitosis就可以了。
<!-- more -->
如果你是用源码安装的，那么就麻烦一点了。首先删除/usr/local/bin下的文件或是/bin/下的文件
```
sudo rm gitosis*
sudo rm easy*
```
然后，到～/repositories目录下，删除gitosis-admin.git目录
```
sudo rm -fr  gitosis-admin.git
```
最关键的是，你需要编辑用来安装gitosis的用户home目录下的一些东西，假如我用git这个用户安装，那么
```
su – git
vi .ssh/authorized_keys
```
把含有gitosis相关的所有全部删除就ok了。