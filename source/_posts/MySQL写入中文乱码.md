title: MySQL写入中文乱码
date: 2015-09-07 22:09:26
categories: MySQL
tags: MySQL
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/mysql.png
---
####MySQL写入中文乱码
<!-- more -->
 MySQL数据库出现中文乱码的原因一般是客户端、服务器、结果集、数据库的字符集不统一造成的，但不是这些都统一就不乱，其中在使用wamp的MySQL时会发现这些都统一还是乱码，全部统一为utf8,这时通过```show variables like "%char%";``
```
mysql> show variables like "%char%";
+--------------------------+-----------------------------------------------+
| Variable_name            | Value                                         |
+--------------------------+-----------------------------------------------+
| character_set_client     | utf8                                          |
| character_set_connection | utf8                                          |
| character_set_database   | utf8                                          |
| character_set_filesystem | binary                                        |
| character_set_results    | utf8                                          |
| character_set_server     | latin1                                        |
| character_set_system     | utf8                                          |
| character_sets_dir       | f:\wamp\bin\mysql\mysql5.6.17\share\charsets\ |
+--------------------------+-----------------------------------------------+
8 rows in set
```
其中```character_set_server     | latin1   ```不统一，通过这条语句修改：set character_set_server='utf8'; 情况不同，这条命令无法修改，```character_set_server```如果默认为latin1要永久更改，用set命令是行不通的，所以更改my.ini文件，在[mysqld]段落增加：```character_set_server=utf8 ```重启MySQL服务即可。 
