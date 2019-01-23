title: 解决Mac os ssh登陆CentOS中文乱码方法
date: 2015-10-19 16:58:49
categories: Mac
tags: [ssh,CentOS,Mac]
---
#### 解决Mac OS ssh登陆CentOS中文乱码方法
<!-- more -->
这种情况一般是终端和服务器的字符集不匹配，MacOSX下默认的是utf8字符集,所以编辑 /etc/ssh_config，注释此行“ SendEnv LANG LC_*”即可，
``` bash
sudo vi /etc/ssh_config，
```
注释此行
``` bash
SendEnv LANG LC_”
```