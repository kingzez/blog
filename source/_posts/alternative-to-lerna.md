---
title: Lerna 的另外一种选择
date: 2020-09-22 15:20:17
tags: [lerna, yarn, monorepo]
---

> 想法来自 vue-next commit [build: remove lerna](https://github.com/vuejs/vue-next/commit/cd5ba7cfcc5bc56392c293422188225cf42b9062?branch=cd5ba7cfcc5bc56392c293422188225cf42b9062&diff=split)

想要通过 monorepo 管理项目，Lerna 是一种选择，Lerna 提供了很多方便的脚本，但是对于一些场景 Lerna 是一种选择，但不是必须的，就像 vue-next 中弃用 Lerna 一样，通过以下几种配置也是可以达到 monorepo 的效果：<br />

- 对于 TypeScript，使用 `tsconfig.json` 中的 `compileOptions.path` 
- 对于 Jest，使用 `jest.config.js` 中的 `moduleNameMapping` 
- 对于 Node.js，使用的是 [Yarn Workspaces](https://yarnpkg.com/blog/2017/08/02/introducing-workspaces/)，在 package.json 中加入 workspaces，比如
```javascript
{
  "name": "demo",
  "devDependencies": {
    "vue": "3.0.0"
  },
  "workspaces": [
    "packages/*"
  ]
}
```


> [https://github.com/vuejs/vue-next/commit/cd5ba7cfcc5bc56392c293422188225cf42b9062?branch=cd5ba7cfcc5bc56392c293422188225cf42b9062&diff=split](https://github.com/vuejs/vue-next/commit/cd5ba7cfcc5bc56392c293422188225cf42b9062?branch=cd5ba7cfcc5bc56392c293422188225cf42b9062&diff=split)
> [https://classic.yarnpkg.com/blog/2017/08/02/introducing-workspaces/](https://classic.yarnpkg.com/blog/2017/08/02/introducing-workspaces/)
