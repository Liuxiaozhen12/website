---
slug: /chaos-engineering-breaking-things-intention
title: Chaos Engineering-故意打破物品
author: 天化的危险度
author_url: https://www.linkedin.com/in/manishdangi/
author_image_url: https://avatars1.githubusercontent.com/u/43807816?s=400
image: /img/chaos-engineering2.png
tags:
  - Chaos工程
  - Chaos Mesh
  - 开源
---

![Chaos-Engineering-Breaking-things-有意使用](/img/chaos-engineering2.png)

“必要性是发明之母”；同样，Netflix不仅仅是网上媒体流畅的平台。 Netflix因为需要建造Chaos工程而诞生。

<!--truncate-->

2008年，Netflix [经历了严重的数据库损坏](https://about.netflix.com/en/news/completing-the-netflix-cloud-migration)。 他们三天内无法投送DVD。 这鼓励Netflix工程师考虑他们的单一建筑物迁移到分布式云基建筑。

Netflix的新分布结构由数百种微型服务组成。 向分布式结构迁移解决了他们单独的点故障问题，但也造成了许多其他复杂问题，需要一个更可靠和容错的制度。 目前，Netflix工程师提出了一个创新的想法，测试系统的故障容忍度，而不会影响客户服务。

他们创建 [Chaos Monkey](https://github.com/Netflix/chaosmonkey)：一个在不同地点造成随机故障的工具，时间间隔不同。 随着Chaos Monkey的发展，出现了一个新的纪律：Chaos Engineering。

“Chaos Engineering是对系统进行试验的学科，以便建立对系统承受生产动荡状况能力的信心。” - [Chaos 原则](https://principlesofchaos.org/)

Chaos Engineering是一种通过应用一系列经验探索来学习你的系统如何运作的方法。 正如科学家为研究物理和社会现象而进行的实验一样。 Chaos Engineering利用实验学习某一特定系统——系统的可靠性、稳定性和在意外或不稳定条件下生存的能力。

当我们有一个大型分布系统时，失败可能是应用失败等若干因素造成的。 基础设施故障、依赖故障、网络故障等等。 这些故障不可能全部被传统方法所涵盖，如整合测试或单元测试，这使Chaos Engineering成为必要的：

- 提高系统的复原力
- 揭露系统的隐藏威胁和脆弱性
- 在系统弱点导致生产失败之前找出这些弱点：

许多人认为他们与Netflix和其他技术巨人相比不那么大。 它们也没有这么大规模的数据库或系统。

它们可能是对的，但在这段时间里，Chaos工程已经发生了很大的变化，它不再局限于Netflix这样的数字公司。 为了确保其系统的连续性和持续可用性，来自不同行业的越来越多的公司正在进行混乱试验。

## Chaos-Mesh

测试 [TiDB](https://pingcap.com/products/tidb)的复原力和可靠性， 工程师 [PingCAP](https://pingcap.com/) 为Chaos 测试开发了一个很棒的工具，叫做 [Chaos Mesh](https://chaos-mesh.org/)一个云层土生土长的Chaos Engineering play, 它会在Kubernetes 环境中引发混乱. Chaos Mesh考虑到分布式系统可能存在的错误，它涵盖了pod、网络、系统I/O和内核。

Chaos Mesh 提供了许多故障注入方法：

- **时钟扭曲：** 模拟时钟
- **容器击杀：** 模拟正在被杀死的容器。
- **cpu-burn:** 模拟CPU压
- **io-attribtion-override:** 模拟文件异常
- **io-default:** 模拟文件系统 I/O 错误
- **io-延迟：** 模拟文件系统I/O 延迟
- **内核注入：** 模拟内核失败
- **内存燃烧：** 模拟内存压力。
- **网络损坏：** 模拟网络数据包损坏
- **网络重复：** 模拟网络数据包
- **网络延迟：** 模拟网络延迟
- **网络损失：** 模拟网络损失
- **网络分区：** 模拟网络分区
- **pod-failment:** 模拟Kubernetes Pods 持续不可用
- **pod-kill:** 模拟正在杀死的 Kubernetes Pod

Chaos Mesh主要侧重于如何简单地进行所有混乱测试，任何使用者都可以轻易地理解。

最近的 [1.0 版本](https://chaos-mesh.org/blog/chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-easier/) 提供了Chaos Dashboard的一般可用性，Chaos简化了Chaos 实验的复杂性。 在点击几个鼠标后，您可以定义Chaos实验的范围，并指定注入chaos的类型， 定义调度规则，并观察Chaos Mesh系统仪表盘中的混乱测试结果。

如果您想在浏览器中尝试Chaos Mesh，请签出 [Katakoda 交互式教程](https://chaos-mesh.org/interactiveTutorial/)， 你可以在哪里获得你的手，甚至不需要部署它。 要了解设计原则和Chaos Mesh如何工作，请阅读 [这个博客](https://chaos-mesh.org/blog/chaos_mesh_your_chaos_engineering_solution) 项目的维护者 [Cwen Yin](https://www.linkedin.com/in/cwen-yin-81985318b/)

## 加入社区

欢迎任何想要探索混乱工程或Chaos Mesh地区的人加入Chaos Mesh社区。 作为Chaos Mesh社区的成员， 我要说，这是一个热爱的社区，在那里，项目维护者热衷于接触和听取你们对改进项目和社区的看法和建议。

要加入并了解更多有关Chaos Mesh的信息，请在 [CNCF 缺少工作区](https://slack.cncf.io/) 中找到 #project-chaos-mesh频道。
