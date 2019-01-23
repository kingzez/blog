title: Git忽略已跟踪文件的改动
date: 2017-04-10 15:13:59
categories: git
tags: git
---

在实际使用中，有时候不可避免会有操作失误，将系统文件添加到版本库中，这时要怎么忽略已跟踪的文件？通过以下步骤：
<!-- more -->
- 从 Git 的数据库中删除对于该文件的追踪；
- 把对应的规则写入 .gitignore，让忽略真正生效；
- commit＋push。
```bash
git update-index --assume-unchanged
```
作用是将此文件移出版本控制，git不会记录此文件的修改，这样再将该文件加到`.ignore`中，与之对应的是
```bash
git update-index --no-assume-unchanged
```
如果是操作失误，即将操作失误移出版本控制的文件重新标识为git可记录的文件