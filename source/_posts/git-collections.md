title: Git使用相关集锦
date: 2017-04-10 15:30:38
categories: git
tags: git
---

总结之前写过的git相关文章，编成目录，也方便自己查看
<!-- more -->

1. [Git创建ssh key](/git创建ssh key/)
2. [git push冲突时解决方法或通过fetch来更新分支代码](/git push冲突时解决方法/)
3. [CentOS下搭建Git服务器](/CentOS下搭建git服务器/)
4. [如何完全删除gitosis](/如何完全删除gitosis/)
5. [人称Git核弹级 filter-branch](/2015/12/20/人称git核弹级 filter-branch/)
6. [git-pull-强制覆盖local](/git-pull-强制覆盖local/)
7. [Git重写最近一次历史](/git重写最近一次历史/)
8. [Git忽略已跟踪文件的改动](/Git-忽略已跟踪文件的改动/)
9. [git stash 用于保存和恢复工作进度](/git-stash-用于保存和恢复工作进度/)

> "git merge" used to allow merging two branches that have no common base by default, which led to a brand new history of an existing project created and then get pulled by an <!--more--> unsuspecting maintainer, which allowed an unnecessary parallel history merged into the existing project. The command has been taught not to allow this by default, with an escape hatch --allow-unrelated-histories option to be used in a rare event that merges histories of two projects that started their lives independently.

```
git merge --allow-unrelated-histories 允许合并无关历史记录的分支，比如两个项目
```