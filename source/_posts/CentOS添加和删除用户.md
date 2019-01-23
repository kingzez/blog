title: CentOS添加和删除用户
date: 2015-10-19 23:00:54
categories: CentOS
tags: CentOS
---
#### 在CentOS下添加和删除用户命令：
<!-- more -->

添加用户 test：
```
adduser test
```
修改test密码：
```
passwd test
```
删除用户test:
```
userdel test
```
删除用户以及用户目录：
```
userdel -f test
```
