---
title: 2018 Flowing Water Review
date: 2019-01-24 01:24:10
tags: review
---

# One

一、众所周知年初各种币火的一塌糊涂，telegram 被称为 ”币圈微信“必不可少会对其搞周边开发，研究 telegram ****相关，大概的需求点是 telegram 群管理机器人，telegram 陌生人拉群，要实现这些势必少不了对 telegram 开放 api  的研究，telegram 的协议很特殊，是独自研发的 MTProto。<!--more-->telegram 好像说的有点多了，对 telegram api 一定了解后，主要入手点是 telegram web 版，看了一下竟然是开源的，而且是使用 angularjs 开发，通过我的魔改，实现了自动拉人与收集群及用户名信息，最终数据是1000+币圈群与100w用户名，当然这些信息本身都是公开的，我只对其进行整理归类。其中使用的技术点有 nodejs，mogodb，angularjs，少量的python。

# Two

二、对微信网页版进行封包，实现登录多个微信，分析网页版微信接口，使用 electronjs 对网页版微信封包，其后客户有 windows xp的需求，由于 electronjs 最低兼容 win7 ，随后又使用了 nwjs 的老版本 0.14.7（当时版本是0.30.x）对聊天的客户端进行的封包，后来又遇到 发送的 mp4 文件无法播放，又对 nwjs 加入对mpeg的编译得以实现播放mp4，总之填了不少坑。其中使用的技术点有 electronjs，nwjs。

# Three

三、接触了 vue，使用 vue-cli 搭建项目，使用 express，mongoDB 完成接口服务，docker 部署上线，独自完成一个项目—代理商平台。

# Four

四、尝试接纳 typescript ，也是因为此前没有深入了解过 ts ，对其语法有一定误解，深入了解后，emm~ 真香，用 ts 重构了之前的项目。

# Five

五、日常更新迭代，没有大的波澜。

# Seven

七、要实现一个 SSO（Single sign-on）单点登录服务，经过一番折腾，对 oauth2 了解了一番，最终使用 oauth2 中的 authorization code 模式实现了单点登录服务，服务架构包括 oauth2-server（认证服务），permission-server（权限中心服务），permission-ui（权限管理前端），server 端统一采用 ts，express，postgresql，权限采用 RBAC（Role-Based Access Control） 模型。

# Eight

八、赶在 vue-cli 3正式版刚发布，新项目尝鲜使用，重构了权限管理的前端。日常更新迭代，没有大的波澜。

# Nine

九、引入 easy-mock，私有化部署，培训 easy-mock 使用。日常更新迭代，没有大的波澜。

# Ten

十、官网升级改版。日常更新迭代，没有大的波澜。

# Eleven

十一、繁忙的。公司内部进销存平台的构建，爬虫服务，url提取工具。server 端新的尝试，使用 koa 替换了 express。其中使用的技术点有koa， electronjs，ts，puppeteer，websocket。

# Twelve

十二、寒冬已至，是个凝重又忧桑的月份，不少小伙伴们走了，然后，然后我接手了小伙伴们的项目……