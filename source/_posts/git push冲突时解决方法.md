title: git push冲突时解决方法
date: 2015-08-23 21:31:42
categories: git
tags: git
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/git-push.png
---
#### `git push`冲突时解决方法
<!-- more -->
在使用git中多人同时修改一个文件时，比如A修改test文件并`push`到远程仓库，B也修改test文件并`push`到远程，这时B`git push`会提示相应的冲突错误。  

解决方法：

* 第一步：从项目仓库中，创建一个新的分支，并测试合并请求中的改动

```
git fetch origin 
```

* 第二步: 创建本地分支local用来存放远程仓库的最新分支

```
git checkout -b local origin/develop
```

* 第三步：进行合并，手动修改合并后的文件

```
git merge develop
```

* 第四步：修改后按照正常的git流程进行操作

```
git add -A
git commit -m "latest"
git push 
```
OK~