title: Mac下brew安装运行redis
date: 2015-09-23 22:32:26
categories: redis
tags: [redis,Mac]
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/redis.png
---
打开终端
``` bash
brew install redis
```
<!-- more -->
如下
``` bash
~ w$ brew install redis
==> Downloading https://homebrew.bintray.com/bottles/redis-3.0.0.yosemite.bottle
######################################################################## 100.0%
==> Pouring redis-3.0.0.yosemite.bottle.tar.gz
==> Caveats
To have launchd start redis at login:
    ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
Then to load redis now:
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
Or, if you don't want/need launchctl, you can just run:
    redis-server /usr/local/etc/redis.conf
==> Summary
🍺  /usr/local/Cellar/redis/3.0.0: 9 files, 892K

```
安装成功，启动redis
``` bash
redis-server
```
如下
``` bash
~ w$ redis-server
3002:C 23 Sep 22:29:50.895 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
3002:M 23 Sep 22:29:50.897 * Increased maximum number of open files to 10032 (it was originally set to 2560).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 3002
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

3002:M 23 Sep 22:29:50.898 # Server started, Redis version 3.0.0
3002:M 23 Sep 22:29:50.898 * The server is now ready to accept connections on port 6379

```
服务端启动，测试客户端，打开新的终端窗口
``` bash
~ w$ redis-cli
127.0.0.1:6379> set foo bar
OK
127.0.0.1:6379> get foo
"bar"
```
ok.成功连接服务端，测试成功。

关闭redis
```bash
redis-cli shutdown
```