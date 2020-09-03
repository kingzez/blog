---
title: Homebrew 加速
date: 2020-03-04 00:03:12
tags: homebrew
---

> [https://mirror.tuna.tsinghua.edu.cn/help/homebrew/](https://mirror.tuna.tsinghua.edu.cn/help/homebrew/)

> [https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git)
> 以上两个国内镜像源Tsinghua较快


### 替换默认源地址

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git

brew update
```

添加至 ~/.zshrc，后 source ~/.zshrc
```bash
# brew mirror
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
```

### 复原默认地址

```bash
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git

brew update
```
