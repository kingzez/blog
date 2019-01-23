title: git merge
date: 2018-05-16 14:24:45
categories: git
tags: git
---
> "git merge" used to allow merging two branches that have no common base by default, which led to a brand new history of an existing project created and then get pulled by an unsuspecting maintainer, which allowed an unnecessary parallel history merged into the existing project. The command has been taught not to allow this by default, with an escape hatch --allow-unrelated-histories option to be used in a rare event that merges histories of two projects that started their lives independently.

```
git merge --allow-unrelated-histories 允许合并无关历史记录的分支，比如两个项目
```