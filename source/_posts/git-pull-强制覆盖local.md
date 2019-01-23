title: git pull 强制覆盖本地文件
date: 2017-03-20 22:55:46
categories: git
tags: git
---

git pull 强制覆盖本地文件
<!-- more -->一旦远程主机的版本库有了更新（Git术语叫做commit），需要将这些更新取回本地，这时就要用到`git fetch`命令。`git fetch`命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。
默认情况下，`git fetch`取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。
```bash
git fetch --all
```
```bash
git reset --hard origin/master
```
```bash
git pull
```
Done!😄