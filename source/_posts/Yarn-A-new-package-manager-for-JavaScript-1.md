title: Yarn A new package manager for JavaScript
date: 2016-10-18 21:44:14
categories: js
tags: yarn
---
Facebook 开源了 yarn 这个新的 JavaScript 包管理工具，yarn 和 npm 做的是完全一样的事情：作为 nodejs 的包管理工具。既然是一样的事情，那么 yarn 必须有一些优点，才能说服大家去用。根据官方网站的介绍，yarn 有以下六项特点：
<!-- more -->
（2016 年 10 月 11 日）到今天（10月18）八天内达到14934star，这是何等的快速……

> 离线模式（重要）

如果之前已经安装过一个软件包，再次安装时就不用再从网络下载了。
这一点很重要，npm 饱受诟病的一点就是，每次安装依赖，都需要从网络下载一大堆东西，而且是全部重新下载。工程多的时候比较烦人。这下子可以节约大量时间了。

> 依赖关系确定性（重要）

在每一台机器上针对同一个工程安装依赖时，生成的依赖关系顺序和版本是一致的。
就是说我在另一台设备上对同样的工程安装依赖，或者把这台机器工程下的 node_modules 目录删除来重新安装依赖。由于关联依赖中，没有指定版本号的库，发生了版本更新，就会导致再次安装的依赖，其中具体某些软件包的版本是不一致的。 在这种情况下，你会发现原来能够正常运行的程序，忽然变得不能工作或一堆 BUG。
yarn 采用的解决方式是，引入了一个 yarn.lock 文件来应对这个问题。lock 机制在很多包管理中都有用到。例如 ruby 的 rubygems 就会生成 Gemfile.lock.
yarn.lock 会记录你安装的所有大大小小的软件包的具体版本号。只要你不删除 yarn.lock 文件，再次运行 yarn install 时，会根据其中记录的版本号获取所有依赖包。你可以把 yarn.lock 提交到版本库里，这样其他同事签出代码并运行 yarn install 时，可以保证大家安装的依赖都是完全一致的。
这就特别适合大型项目的多人协作开发和部署。

> 更好的网络性能

下载软件包时，会进行更好的排序，避免“请求瀑布”，最大限度提高网络利用率。

> 多注册来源处理

所有的依赖包，不管他被不同的库间接关联引用多少次，安装这个包时，只会从一个注册来源去装，要么是 npm 要么是 bower, 防止出现混乱不一致。

> 网络弹性处理

安装依赖的过程中，不会因为某个单次网络请求的失败导致整个安装挂掉（不由自主的要黑一下 npm）。当请求失败时会进行自动重试。

> 扁平模式（重要）

当关联依赖中包括对某个软件包的重复引用，在实际安装时将尽量避免重复的创建。

为了说明这个问题，我们假设目前工程依赖 A, B, C 三个库，而他们对某个库 somelib 存在这样的依赖关系：


A - somelib 1.4.x
B - somelib 1.6.x
C - somelib 1.6.x

如果要安装 ABC 三个库，那么 somelib 会存在版本冲突。npm 会在实际安装时，给三个库分别下载各自依赖的 somelib 版本。假设 npm 先安装了 A, 由于 A 依赖 somelib 1.4.x 版本，那么 1.4.x 会变成主版本。

再安装 B, C 时，由于 B, C 依赖的都不是 1.4.x, 于是安装完之后，关系就变成这个样子了：

node_modules
├── A
├── somelib 1.4.x
├── B
│  └── node_modules
│      └── somelib 1.6.x
└── C
    └── node_modules
        └── somelib 1.6.x

明明 B, C 都依赖 1.6.x 版本，实际上 npm 却要把这个版本保存两次，这样明显是对磁盘空间的浪费。我们把这种情况就称为“不扁平的”。尽管 npm 也提供了诸如 flat 等指令去支持“扁平化”处理，yarn 明显试图在这方面做得更好。

总结三大特点： 超快速、超级安全、超级可靠

（有没有感觉这猫忒怪……）

下面说一下安装使用：
yarn与npm相同，都是依靠package.json文件安装包。yarn要怎么安装呢？

macOS下：
首先需要先安装Node.js，如果已安装则略过。通过Homebrew安装
```
brew update
brew install yarn
```
设置PATH
需要终端中设置PATH环境变量，以便于全局使用。
添加`export PATH="$PATH:$HOME/.yarn/bin" `到 profile中 (可能是 .profile, .bashrc, .zshrc, 等等)
测试一下~
yarn --version

Windows下：
首先需要先安装Node.js，如果已安装则略过。windows下有两种方式安装
1.下载.msi文件安装 https://yarnpkg.com/latest.msi
2.通过Chocolatey安装（chocolatey是windows下包管理软件，详情移步了解https://chocolatey.org/install）
```
choco install yarn
```
设置PATH
添加`set PATH=%PATH%;C:\.yarn\bin` 到shell环境变量中

npm下快速安装
可以通过npm安装，非常方便
npm install -g yarn

设置PATH
Unix/Linux/macOS
需要终端中设置PATH环境变量，以便于全局使用。
添加 `export PATH="$PATH:$HOME/.yarn/bin" `到 profile中 (可能是 .profile, .bashrc, .zshrc, 等等)
windows
添加`set PATH=%PATH%;C:\.yarn\bin `到shell环境变量中
测试一下~
```
yarn --version
```
安装之后，就可以愉快优雅的创建项目了~

创建一个新项目

```
yarn init
```
添加项目依赖包

```
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]
```
更新依赖包

```
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```
移除依赖包

```
yarn remove [package]
```
安装项目依赖的所有
```
yarn
```
或者

```
yarn install
```
简单比较一下yarn与npm的命令
运行`yarn`运行相当于`npm install`
运行  `yarn add <name>`运行相当于`npm install --save <name>`

Reference：
https://yarnpkg.com/

https://github.com/yarnpkg/yarn

https://code.facebook.com/posts/1840075619545360

https://zhuanlan.zhihu.com/p/22892675?from=timeline&isappinstalled=0

https://www.zhihu.com/question/51502849
