---
title: 了解什么是微前端
date: 2019-07-04 16:16:42
tags: [微前端, 'micro frontend']
---

作为前端开发人员，这些年来你一直在开发单体应用，即使你已经知道这是一个不好的做法。 您将代码划分为组件，使用 *require* 或 *import* 并将package.json中定义的npm包或已安装的子git仓库添加到项目中，但最终构建了一个整体。 是时候改变它了。

### 为什么你的代码是一个单体？

除了已经实现了微前端的应用之外，所有前端应用本质上都是单一的应用。 原因是如果您正在使用 React 库进行开发，并且如果您有两个团队，则两个团队都应该使用相同的React 库，并且两个团队应该在部署时保持同步，并且在代码合并期间始终会发生冲突。 它们没有完全分离，很可能它们维护着相同的仓库并具有相同的构建系统。 单体应用的退出被标志为微服务的出现。 但是它适用于后端！😱

### 什么是微服务？

对于微服务，一般而言最简单的解释是，它是一种开发技术，允许开发人员为平台的不同部分进行独立部署，而不会损害其他部分。 独立部署的能力允许他们构建孤立或松散耦合的服务。 为了使这个体系结构更稳定，有一些规则要遵循，可以总结如下：每个服务应该只有一个任务，它应该很小。 所以负责这项服务的团队应该很小。 关于团队和项目的规模，[James Lewis 和 Martin Fowler](https://martinfowler.com/articles/microservices.html#HowBigIsAMicroservice) 在互联网上做出的最酷解释之一如下：

> 在我们与微服务从业者的对话中，我们看到了一系列服务规模。 报道的最大规模遵循亚马逊关于Two Pizza Team的概念（即整个团队可以由两个比萨饼供给），意味着不超过十几个人。 在规模较小的规模上，我们已经看到了一个由六人组成的团队支持六项服务的设置。


我画了一个简单的草图，为整体和微服务提供了直观的解释：

![](https://user-images.githubusercontent.com/10891613/60702523-c2af6800-9f31-11e9-988e-598f1c66f423.jpg)


从上图可以理解，微服务中的每个服务都是一个独立的应用，除了UI。 UI仍然是一体的！ 当一个团队处理所有服务并且公司正在扩展时，前端团队将开始苦苦挣扎并且无法跟上它，这是这种架构的瓶颈。

![](https://user-images.githubusercontent.com/10891613/60702910-d14a4f00-9f32-11e9-9879-0dc0510ac3ef.jpg)

除了瓶颈之外，这种架构也会导致一些组织问题。 假设公司正在发展并将采用需要 *跨职能* 小团队的敏捷开发方法。 在这个常见的例子中，产品所有者自然会开始将故事定义为前端和后端任务，而 *跨职能* 团队将永远不会成为真正的 *跨职能* 部门。 这将是一个浅薄的泡沫，看起来像一个敏捷的团队，但它将在内部分开。 关于管理这种团队的更多信息将是一项非常重要的工作。 在每个计划中，如果有足够的前端任务或者sprint中有足够的后端任务，则会有一个问题。 为了解决这里描述的所有问题和许多其他问题，几年前出现了**微前端**的想法并且开始迅速普及。

### 解决微服务中的瓶颈问题：Micro Frontends🎉

解决方案实际上非常明显，采用了多年来为后端服务工作的相同原则：将前端整体划分为小的UI片段。 但UI与服务并不十分相似，它是最终用户与产品之间的接口，应该是一致且无缝的。 更重要的是，在单页面应用时代，整个应用在客户端的浏览器上运行。 它们不再是简单的HTML文件，相反，它们是复杂的软件，达到了非常复杂的水平。 现在我觉得微型前端的定义是必要的：

> Micro Frontends背后的想法是将网站或Web应用视为独立团队拥有的功能组合。 每个团队都有一个独特的业务或任务领域，做他们关注和专注的事情。团队是跨职能的，从数据库到用户界面开发端到端的功能。（[micro-frontends.org](https://micro-frontends.org/)）

根据我迄今为止的经验，对于许多公司来说，直接采用上面提出的架构真的很难。 许多其他人都有巨大的遗留负担，这使他们无法迁移到新的架构。 出于这个原因，更柔软的中间解决方案更加灵活，易于采用和安全迁移至关重要。 在更详细地概述了体系结构后，我将尝试提供一些体系结构的洞察，该体系结构确认了上述提议并允许更灵活的方式。 在深入了解细节之前，我需要建立一些术语。

### 整体结构和一些术语

让我们假设我们通过业务功能垂直划分整体应用结构。 我们最终会得到几个较小的应用，它们与单体应用具有相同的结构。 但是如果我们在所有这些小型单体应用之上添加一个特殊应用，用户将与这个新应用进行通信，它将把每个小应用的旧单体UI组合成一个。 这个新图层可以命名为**拼接图层**，因为它从每个微服务中获取生成的UI部件，并为最终用户组合成一个*无缝* UI，这将是微前端的最直接实现🤩

![](https://user-images.githubusercontent.com/10891613/60702748-5aad5180-9f32-11e9-93f4-eaaf99415697.jpg)

为了更好地理解，我将每个小型单体应用称为**微应用**，因为它们都是独立的应用，而不仅仅是微服务，它们都有UI部件，每个都代表端到端的业务功能。

众所周知，今天的前端生态系统功能多样，而且非常复杂。 因此，当实现真正的产品时，这种直接的解决方案还不够。

### 要解决的问题

虽然这篇文章只是一个想法，但我开始使用Reddit讨论这个想法。 感谢社区和他们的回复，我可以列出一些需要解决的问题，我将尝试逐一描述。

*当我们拥有一个完全独立的独立**微应用**时，如何创建无缝且一致的UI体验？*

好吧，这个问题没有灵丹妙药的答案，但其中一个想法是创建一个共享的UI库，它也是一个独立的微应用。 通过这种方式，所有其他**微应用**将依赖于共享的UI库微应用。 在这种情况下，我们刚刚创建了一个共享依赖项，我们就杀死了独立**微应用**的想法。

另一个想法是在根级共享CSS自定义变量（ [CSS custom variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables) ）。 此解决方案的优势在于应用之间的全局可配置主题。

或者我们可以简单地在应用团队之间共享一些SASS变量和混合。 这种方法的缺点是UI元素的重复实现，并且应该对所有**微应用**始终检查和验证类似元素的设计的完整性。

*我们如何确保一个团队不会覆盖另一个团队编写的CSS？*

一种解决方案是通过CSS选择器名称进行CSS定义，这些名称由微应用名称精心选择。 通过将该范围任务放在**拼接层**上将减少开发开销，但会增加**拼接层**的责任。

另一种解决方案可以是强制每个微应用成为自定义Web组件（[custom web component](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)）。 这个解决方案的优点是浏览器完成了范围设计，但需要付出代价：使用shadow DOM进行服务器端渲染几乎是不可能的。 此外，自定义元素没有100％的[浏览器支持](https://caniuse.com/#search=custom%20elements)，特别是IE。

*我们应该如何在微应用之间共享全局信息？*

这个问题指出了关于这个主题的最关注的问题之一，但解决方案非常简单：HTML 5具有相当强大的功能，大多数前端开发人员都不知道。 例如，**自定义事件（custom events）** 就是其中之一，它是在微应用中共享信息的解决方案。

或者，任何共享的pub-sub实现或T39可观察的实现都可以实现。 如果我们想要一个更复杂的全局状态处理程序，我们可以实现共享的微型Redux，通过这种方式我们可以实现更多的相应式架构。

*如果所有微应用都是独立应用，我们如何进行客户端路由？*

这个问题取决于设计的每个实现, 所有主要的现代框架都通过使用浏览器历史状态在客户端提供强大的路由机制, 问题在于哪个应用负责路由以及何时。

我目前的实用方法是创建一个共享客户端路由器，它只负责顶级路由，其余路由器属于相应的微应用。 假设我们有 `/content/:id` 路由定义。 共享路由器将解析 `/content`，已解析的路由将传递到ContentMicroApp。 ContentMicroApp是一个独立的服务器，它将仅使用 `/:id` 进行调用。

*我们必须是服务器端渲染，但是有可能使用微前端吗？*

服务器端呈现是一个棘手的问题。 如果你正在考虑iframes缝合**微应用**然后忘记服务器端渲染。 同样，拼接任务的Web组件也不比iframe强大。 但是，如果每个**微应用**能够在服务器端呈现其内容，那么**拼接层**将仅负责连接服务器端的HTML片段。

*与传统环境集成至关重要！ 但是怎么样？*

为了整合遗留系统，我想描述我自己的策略，我称之为“ *渐进式入侵* ”。

首先，我们必须实现拼接层，它应该具有透明代理的功能。 然后我们可以通过声明一个通配符路径将遗留系统定义为**微应用**：*LegacyMicroApp* 。 因此，所有流量都将到达拼接层，并将透明地代理到旧系统，因为我们还没有任何其他微应用。

下一步将是我们的 *第一次逐步入侵* ：我们将从LegacyMicroApp中删除主要导航并用依赖项替换它。 这种依赖关系将是一个使用闪亮的新技术实现的**微应用**：*NavigationMicroApp* 。

现在，拼接层将每个路径解析为 *Legacy Micro App* ，它将依赖关系解析为 *Navigation MicroApp* ，并通过连接这两个来为它们提供服务。

然后通过主导航遵循相同的模式来为引导下一步。

然后我们将继续从Legacy MicroApp中获取逐步重复以上操作，直到没有任何遗漏。

*如何编排客户端，这样我们每次都不需要重新加载页面？*

**拼接层**解决了服务器端的问题，但没有解决客户端问题。 在客户端，在将已粘贴的片段作为无缝HTML加载后，我们不需要每次在URL更改时加载所有部分。 因此，我们必须有一些异步加载片段的机制。 但问题是，这些片段可能有一些依赖关系，这些依赖关系需要在客户端解决。 这意味着微前端解决方案应提供加载**微应用**的机制，以及依赖注入的一些机制。

---

根据上述问题和可能的解决方案，我可以总结以下主题下的所有内容：

**客户端**

- 编排
- 路由
- 隔离微应用
- 应用之间通信
- 微应用UI之间的一致性

**服务端**

- 服务端渲染
- 路由
- 依赖管理

### 灵活、强大而简单的架构

![](https://user-images.githubusercontent.com/10891613/60702255-f938b300-9f30-11e9-80c2-5e2470b6edcd.jpg)

所以，这篇文章还是很值得期待的！ 微前端架构的基本要素和要求终于显现！

在这些要求和关注的指导下，我开始开发一种名为microfe的解决方案。 😎在这里，我将通过抽象的方式强调其主要组件来描述该项目的架构目标。

它很容易从客户端开始，它有三个独立的主干结构：*AppsManager，* *Loader，* *Router* 和一个额外的*MicroAppStore。*

![](https://user-images.githubusercontent.com/10891613/60702174-b4ad1780-9f30-11e9-9bdb-d3baefd54981.jpg)

### AppsManager

AppsManager 是客户端微应用编排的核心。 AppsManager的主要功能是创建依赖关系树。 当解决了微应用的所有依赖关系时，它会实例化微应用。

### Loader

客户端微应用编排的另一个重要部分是Loader。 加载器的责任是从服务器端获取未解析的微应用。

### Router

为了解决客户端路由问题，我将 Router 引入了 ***microfe***。 与常见的客户端路由器不同，***microf*** 的功能有限，它不解析页面而是微应用。 假设我们有一个URL `/content/detail/13` 和一个ContentMicroApp。 在这种情况下，***microfe*** 将URL解析为 `/content/`，它将调用ContentMicroApp `/detail/13` URL部分。

### MicroAppStore

为了解决微应用到微应用客户端的通信，我将MicroAppStore引入了 ***microfe***。 它具有与Redux库类似的功能，区别在于：它对异步数据结构更改和reducer 声明更灵活。

---

服务器端部分在实现上可能稍微复杂一些，但结构更简单。 它只包含两个主要部分 *StitchingServer* 和许多*MicroAppServer。*

### MicroAppServer

![](https://user-images.githubusercontent.com/10891613/60702407-88de6180-9f31-11e9-9698-8fd841967671.jpg)

*MicroAppServer* 的最小功能可以概括为 *init* 和 *serve。*

虽然 *MicroAppServer* 首先启动它应该做的是使用 *微应用声明* 调用 *SticthingServer* 注册端点，该声明定义了 *MicroAppServer* 的微应用 *依赖关系，* *类型* 和 *URL架构。* 我认为没有必要提及服务功能，因为没有什么特别之处。

### StitchingServer

![](https://user-images.githubusercontent.com/10891613/60702852-a52ece00-9f32-11e9-9327-0b64a43b9dda.jpg)

*StitchingServer* 为 *MicroAppServers* 提供注册端点。 当 *MicroAppServer* 将自己注册到 *StichingServer* 时，*StichingServer* 会记录*MicroAppServer* 的声明。

稍后，*StitchingServer* 使用声明从请求的URL解析 *MicroAppServers。*

解析M *icroAppServer* 及其所有依赖项后，CSS，JS和HTML中的所有相对路径都将以相关的 *MicroAppServer* 公共URL为前缀。 另外一步是为CSS选择器添加一个唯一的 *MicroAppServer* 标识符，以防止客户端的微应用之间发生冲突。

然后 *StitchingServer* 的主要职责就是：从所有收集的部分组成并返回一个无缝的HTML页面。

### 其他实现一览

甚至在2016年被称为微前端之前，许多大公司都试图通过 [BigPipe](https://www.facebook.com/notes/facebook-engineering/bigpipe-pipelining-web-pages-for-high-performance/389414033919/) 来解决Facebook等类似问题。 如今这个想法正在获得验证。 不同规模的公司对该主题感兴趣并投入时间和金钱。 例如，Zalando开源了其名为Project Mosaic的解决方案。 我可以说，微型和 [Project Mosaic](https://www.mosaic9.org/).遵循类似的方法，但有一些重要的区别。 虽然microfe采用完全分散的路由定义来增强每个微应用的独立性，但Project Mosaic更喜欢每条路径的集中路由定义和布局定义。 通过这种方式，Project Mosaic可以实现轻松的A/B测试和动态布局生成。

对于该主题还有一些其他方法，例如使用iframe作为拼接层，这显然不是在服务器端而是在客户端。 这是一个非常简单的解决方案，不需要太多的服务器结构和DevOps参与。 这项工作只能由前端团队完成，因此可以减轻公司的组织负担，同时降低成本。

已经有一个框架叫做 *[single-spa](https://single-spa.js.org/)。* 该项目依赖于每个应用的命名约定来解析和加载微应用。 容易掌握想法并遵循模式。 因此，在您自己的本地环境中尝试该想法可能是一个很好的初步介绍。 但是项目的缺点是你必须以特定的方式构建每个微应用，以便他们可以很好地使用框架。

### 最后的想法

我相信微前端话题会更频繁地讨论。 如果该主题能够引起越来越多公司的关注，它将成为大型团队的事实发展方式。 在不久的将来，任何前端开发人员都可以在这个架构上掌握一些见解和经验，这真的很有用。

### 考虑贡献

我正在大力尝试微前端，在我的脑海中有一个崇高的目标：创建一个微框架框架，可以解决大多数问题，而不会影响性能，易于开发和可测试性。 如果您有任何明智的想法，请不要犹豫，访问我的repo，打开问题或通过下面的评论或 Twitter DM 与我联系。 我会在那里帮助你！🙂

翻译：Vincent.W
原文：[Understanding Micro Frontends](https://hackernoon.com/understanding-micro-frontends-b1c11585a297)