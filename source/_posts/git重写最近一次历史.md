title: Git重写最近一次历史
date: 2017-04-10 15:01:56
categories: git
tags: git
---

假设你刚刚提交的一个commit中有个错误，想修改一下commit的message，但是又不想在修改无意义的文件再次commit，这时需要使用命令`--amend`即重写最近的一次commit历史。
<!-- more -->
```bash
git commit --amend
```
或者
```bash
git commit --amend -m "your new commit message"
```
在amend之后，commit的时间不会被重写，如果想要重写commit的时间需要使用
```bash
git commit --amend --reset-author
```
其中需要注意的是amend命令重写上一个commit，所以在已经push了上一个commit的情况下，尽可能的不要使用amend。如果你执意要amend已经push的commit，一定要保证这个commit所在的分支只有你一个人，否则将给其他人带来毁灭性的打击，在amend之后使用`git push --force`。