title: iterm 2 cd 神器
date: 2016-07-26 23:44:32
categories: Mac
tags: iterm2
---
*autojump - 更快的方式来浏览你的文件系统*
<!-- more -->
安装通过Homebrew

```
brew install autojump
```
在zsh下需要配置zsh文件，在.zshrc下添加命令
```
$[[ -s `brew --prefix`/etc/autojump.sh ]] && . `brew --prefix`/etc/autojump.sh
```
保存文件，即可使用。
使用方法是`j `经常使用或之前使用过的文件夹，即可直接跳到你要浏览的目录，省去中间需要的各种`cd`。非常方便。

