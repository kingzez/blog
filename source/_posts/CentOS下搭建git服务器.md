title: CentOS下搭建git服务器
date: 2015-10-18 23:46:02
categories: git
tags: [git,CentOS]
photos:
- http://7xkghm.com1.z0.glb.clouddn.com/image/blog/centos-gitserver.png
---
#### CentOS下搭建git服务器
1. 通过源码编译安装，git-1.9.5.tar.gz安装git

<!-- more -->

```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
wget http://git-core.googlecode.com/files/git-1.9.5.tar.gz
tar zxvf git-1.9.5.tar.gz
cd git-1.9.5
make prefix=/usr/local all
make prefix=/usr/local install  

```
root用户运行
查看版本号：git --version
2. 安装gitosis需要安装python，python-setuptools依赖

```
yum install python python-setuptools
git clone git://github.com/res0nat0r/gitosis.git
cd gitosis
python setup.py install
```
3. 在客户端开发本上生成公共密钥（必须）·用来初始化gitosis

```
ssh-keygen -t rsa #不需要密码,一路回车就行(在本地操作)
scp ~/.ssh/id_rsa.pub root@xxx:/tmp/ # 上传你的ssh public key到服务器
```
4. 初始化gitosis[服务器端]

```
adduser git # 新增一个git用户(先添加用户组　groupadd git)
su git # 切换倒git用户下
gitosis-init < /tmp/id_rsa.pub # id_rsa.pub是刚刚传过来的,注意放在/tmp目录主要是因为此目录权限所有人都有定权限的
rm /tmp/id_rsa.pub # id_rsa.pub已经无用，可删除.
```
5. 修改post-update权限(使客户端能访问)

```
sudo chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update
```
6. 获取并配置gitosis-admin [客户端]

```
git clone git@xxx:gitosis-admin.git  # 切换到root用户并在本地执行，获取gitosis管理项目，将会产生一个gitosis-admin的目录，里面有配置文件gitosis.conf和一个 keydir 的目录，keydir目录主要存放git用户名
vi gitosis-admin/gitosis.conf  # 编辑gitosis-admin配置文件
```
如果无法git clone的话，可以使用`git clone git@xxx:/home/git/repositories/gitosis-admin.git`

```
* 在gitosis.conf底部增加
[group 组名]
writable = 项目名
members = 用户  # 这里的用户名字 要和 keydir下的文件名字相一致
* vi下按ZZ（大写）两次会执行自动保存并退出，完成后执行
cd gitosis-admin
git add .
git commit -a -m “xxx xx” # 要记住的是，如果每次添加新文件必须执行git add .，或者git add filename，如果没有新加文件，只是进行修改的话就可以执行此句。
* 修改了文件以后一定要PUSH到服务器，否则不会生效。
git push
```
7. 新建项目
到此步gitosis基本已经配置完成，接下来新建项目到服务器的操作，向配置文件gitosis.
conf添加项目信息

```
[group git-test] # 组名称
writable = git-test # 项目名称
members = xxx      # 用户名xxx一定要与客户端使用的用户名完全一样，否则无权限操作
```
`注意的是每次修改必须更新到git server服务器`
```
git commit -a -m “添加新项目git-test，新项目的目录是git-test，该项目的成员是xxx“ # “”里的内容自定
git push
```
8. 回到客户端开发本上创建项目git-test

```
mkdir /home/user/git-test
cd /home/user/git-test
git init 
touch README.md
git add .
git commit -a -m "initalize git-test"
```
然后将新建项目放到git server服务器上
```
git remote add origin git@xxx:git-test.git
git push origin master /or/ git push -u origin master
```
完成后会在终端中看到
`gitosis-admin  gitosis-admin.git  git-test.git`

 **!@!#!@** 遇到的问题
`ERROR:gitosis.serve.main:Repository read access denied`
原因:gitosis.conf中的members与keydir中的用户名不一致，如gitosis中的members = Macbook@sth，但keydir中的公密名却叫Macbook.pub
解决:使keydir的名称与gitosis中members所指的名称一致。 改为members = Macbook 或 公密名称改为Macbook@sth.pub 
