title: Mac OSX 10.10下使用Homebrew安装MongoDB及启动连接
date: 2015-07-16 17:45:18
categories: Mac
tags: [Mac,homebrew,mongodb]
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/install-mongodb.png
---
Homebrew是Mac OSX下一个包依赖管理工具，用它来安装软件非常的方便只需要brew install ***，不用再操心安装过程中软件的依赖问题。
<!--more-->
###Homebrew 安装

``` 
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
```

###MongoDB 安装

``` 
brew install mongodb
```
等待安装，我的是之前下载过所以直接安装
``` bash
~ w$brew install mongodb
==> Downloading https://homebrew.bintray.com/bottles/mongodb-3.0.2.yosemite.bott
Already downloaded: /Library/Caches/Homebrew/mongodb-3.0.2.yosemite.bottle.tar.gz
==> Pouring mongodb-3.0.2.yosemite.bottle.tar.gz
==> Caveats
To have launchd start mongodb at login:
    ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents
Then to load mongodb now:
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
Or, if you don't want/need launchctl, you can just run:
    mongod --config /usr/local/etc/mongod.conf
==> Summary
🍺  /usr/local/Cellar/mongodb/3.0.2: 17 files, 153M

```
需要测试是否安装成功，用mongod命令进行测试，执行mongod后会返回一顿错误、
``` bash
~ w$mongod
2015-07-16T22:45:19.706+0800 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory /data/db not found., terminating
2015-07-16T22:45:19.706+0800 I CONTROL  [initandlisten] dbexit:  rc: 100

```
错误信息是dbpath /data/db不存在，需要创建/data/db这个目录或者使用--dbpath参数项指定一个已经存在的目录，dbpath /data/db这个目录是用来存储MongoDB的数据文件，既然不存在我们就创建`/data/db`这个目录或者使用`--dbpath`参数项指定一个已经存在的目录。
首先在终端输入`cd /`命令返回到磁盘根目录
然后输入`mkdir -p /data/db`创建`/data/db`
再次输入`mongod`命令启动MongoDB的服务
又返回一堆错误信息

``` bash
/ w$mongod
2015-07-16T22:55:10.232+0800 I STORAGE  [initandlisten] exception in initAndListen: 98 Unable to create/open lock file: /data/db/mongod.lock errno:13 Permission denied Is a mongod instance already running?, terminating
2015-07-16T22:55:10.232+0800 I CONTROL  [initandlisten] dbexit:  rc: 100

```
不能创建/打开/data/db/mongod.lock这个文件，原因是Permission denied（权限拒绝），原因是当前用户执行mongod这个命令时，对/data/db没有操作权限，给/data/db加上权限。
```
sudo chown -R  当前登录的用户名 /data
```
再次在终端输入`mongod`启动MongoDB的服务
``` bash
/ w$sudo chown -R w /data
/ w$mongod
2015-07-16T22:55:49.058+0800 I JOURNAL  [initandlisten] journal dir=/data/db/journal
2015-07-16T22:55:49.059+0800 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
2015-07-16T22:55:49.084+0800 I JOURNAL  [durability] Durability thread started
2015-07-16T22:55:49.084+0800 I JOURNAL  [journal writer] Journal writer thread started
2015-07-16T22:55:49.085+0800 I CONTROL  [initandlisten] MongoDB starting : pid=2473 port=27017 dbpath=/data/db 64-bit host=W-MacBook-Pro.local
2015-07-16T22:55:49.085+0800 I CONTROL  [initandlisten] db version v3.0.2
2015-07-16T22:55:49.085+0800 I CONTROL  [initandlisten] git version: nogitversion
2015-07-16T22:55:49.085+0800 I CONTROL  [initandlisten] build info: Darwin yosemitevm.local 14.3.0 Darwin Kernel Version 14.3.0: Mon Mar 23 11:59:05 PDT 2015; root:xnu-2782.20.48~5/RELEASE_X86_64 x86_64 BOOST_LIB_VERSION=1_49
2015-07-16T22:55:49.085+0800 I CONTROL  [initandlisten] allocator: system
2015-07-16T22:55:49.085+0800 I CONTROL  [initandlisten] options: {}
2015-07-16T22:55:49.086+0800 I INDEX    [initandlisten] allocating new ns file /data/db/local.ns, filling with zeroes...
2015-07-16T22:55:49.423+0800 I STORAGE  [FileAllocator] allocating new datafile /data/db/local.0, filling with zeroes...
2015-07-16T22:55:49.423+0800 I STORAGE  [FileAllocator] creating directory /data/db/_tmp
2015-07-16T22:55:50.063+0800 I STORAGE  [FileAllocator] done allocating datafile /data/db/local.0, size: 64MB,  took 0.639 secs
2015-07-16T22:55:50.412+0800 I NETWORK  [initandlisten] waiting for connections on port 27017
2015-07-16T22:56:05.218+0800 I NETWORK  [initandlisten] connection accepted from 127.0.0.1:49580 #1 (1 connection now open)
```
启动成功了，然后再打开一个新的终端窗口输入`mongo`命令
``` bash
/ w$mongo
MongoDB shell version: 3.0.2
connecting to: test
>
```
OK 安装完成,可以使用。