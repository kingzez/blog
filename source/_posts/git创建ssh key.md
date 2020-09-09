title: Git创建ssh key
date: 2015-08-22 15:37:35
categories: git
tags: git
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/git-ssh.png
---

#### 一、设置Git的username ，email：
```bash
   git config --global user.name "kingzez"
   git config --global user.email "kingzez@sina.com"
```
#### 二、如何针对每个repo单独设定username和email？

有时，你可能想为不同的 repo（比如公司的项目和个人项目）设置不同的 commit 信息，很简单：
```bash
git config user.name "kingzez"
git config user.email "kingzez@sina.com"
```
可以看出，在使用 git config 时，默认修改的是当前 repo 的配置，如果添加 --global 则会修改全局的配置。
<!--more-->
#### 三、创建ssh密匙过程
```bash
     1.查看是否已经有了ssh密钥：cd ~/.ssh
        如果没有密钥则不会有此文件夹，有则不影响，备份删除都可以
     2.生存密钥：
        ssh-keygen -t rsa -C “kingzez@sina.com”
        3个回车，密码为空。
     3.操作完成生成 id_rsa和id_rsa.pub
```
#### 三、添加ssh key 到github 或coding等。
