---
slug: /庆祝一年一度的连环画展-回顾-未来活动
title: '庆祝Chaos Mesh一年：回顾和展望未来'
author: Cwen Yin, Calvin Weng
author_title: Chaos Mesh 维护者
author_url: https://github.com/chaos-mesh/chaos-mesh/blob/master/MAINTAINERS.md
author_image_url: https://avatars1.githubusercontent.com/u59082378?v=4
image: /img/正在庆祝一年的连环画中查找-前方.jpg
tags:
  - Chaos Mesh
  - Chaos工程
---

![庆祝Chaos Mesh一年：回顾和展望未来](/img/celebrating-one-year-of-chaos-mesh-looking-back-and-ahead.jpg)

Chaos Mesh是首次在 GitHub 上开源的一年了。 Chaos Mesh起初仅仅是一种故障注射工具，现在正朝着建立混乱工程生态的目标迈进。 同时，Chaos Mesh社区也是从零开始建造的，帮助 [Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh) 加入了CNCF作为一个沙盒项目。

<!--truncate-->

在这篇文章中，我们将与你们分享Chaos Mesh在过去一年里的成长和变化。 并讨论其今后的目标和计划。

## 项目：以明确的目标欣欣欣欣向荣。

在过去一年里，Chaos Mesh在社区的共同努力下以令人印象深刻的速度增长。 从第一个版本到最近发布的 [v1.1。](https://github.com/chaos-mesh/chaos-mesh/releases/tag/v1.1.0), Chaos Mesh在功能、易于使用和安全方面有了很大改进。

### 功能

当第一个开源时，Chaos Mesh只支持三种错误类型：PodChaos、NetworkChaos和IOChaos。 仅在一年内，Chaos Mesh 可以在网络内部、系统时钟、JVM 应用、文件系统、操作系统等所有故障注入。

![Chaos 测试](/img/chaos-tests.png)

在不断优化之后，Chaos Mesh现在提供了一个灵活的安排机制，使用户能够更好地设计自己的混乱试验。 这为混乱的组织奠定了基础。

In the meantime, we are happy to see that a number of users have started to [test Chaos Mesh on major cloud platforms](https://github.com/chaos-mesh/chaos-mesh/issues/1182), such as Amazon Web Services (AWS), Google Kubernetes Engine (GKE), Alibaba Cloud, and Tencent Cloud.  我们一直在进行兼容性测试和调整，以支持在特定云端平台</a> 中喷射

个故障。</p> 

为了更好地支持 Kubernetes 原生组件和节点级故障，我们开发了 [Chaosd](https://github.com/chaos-mesh/chaosd)，它提供了物理节点级故障注入。 我们正在广泛测试和改进这个功能，以便在今后几个月内发布。



### 便捷使用

自第一天以来，简化使用手续一直是Chaos Mesh发展的指导原则之一。  您可以使用单个命令行部署Chaos Mesh。 V1.0 版本带来了期待已久的Chaos Dashboard，这是一种一站式网络接口，供用户使用来安排混乱试验。 您可以定义chaos实验的范围，指定Chaos注入类型，定义调度规则。 并观察混乱实验的结果——仅在同一个网络界面中点击几次。

![Chaos 仪表板](/img/chaos-dashboard1.png)

在V1.0之前，许多用户报告在注射IOChaos错误时因各种配置问题而被阻止。 经过紧张的调查和讨论，我们放弃了原先的“侧边机”的实施。 相反，我们使用chaos-daemon来动态地入侵目标Pod，这大大简化了逻辑。 这种优化使得能够在Chaos Mesh中注入动态I/O故障。 用户可以只注重他们的实验，而不必担心其他配置。



### 安全

我们改善了Chaos Mesh的安全。 它现在提供了一套全面的选择器来控制试验的范围，并支持设置具体的命名空间来保护重要的应用。 更有甚者，命名空间权限的支持允许用户将混乱试验的“爆炸辐射”限制在一个特定的命名空间。

此外，Chaos Mesh直接重新使用Kubernetes的原生许可机制，并支持Chaos仪表盘上的核查。 这将保护您免受其他用户的错误影响，可能导致混乱实验失败或无法控制。



## 云层生态系统：一体化与合作

In July 2020, Chaos Mesh was successfully [accepted as a CNCF Sandbox project](https://chaos-mesh.org/blog/chaos-mesh-join-cncf-sandbox-project). 这表明Chaos Mesh最初得到了云端土著社区的承认。 同时， 这意味着Chaos Mesh有一项明确的任务：促进在云层本土地区应用混乱工程，并与其他云层本土项目合作，使我们能够共同成长。



### 格拉萨纳

为了进一步提高Chaos实验的可观察性，我们已经为Chaos Mesh单独设置了一个 [Grafana 插件](https://github.com/chaos-mesh/chaos-mesh-datasource) 。 允许用户在应用程序监测面板上直接显示实时混乱的实验信息。 这样，用户可以同时观察应用程序的运行状态和当前混乱状态的实验信息。



### GitHub Action

使用户能够在开发阶段运行混乱实验， 我们开发了 [chaos-mesh-action](https://github.com/chaos-mesh/chaos-mesh-action) 项目，允许Chaos Mesh在 GitHub 操作流程中运行。 这样，Chaos Mesh就可以很容易地融入系统的日常开发和测试。



### 蒂口克特

[TiPocket](https://github.com/pingcap/tipocket) 是一个自动化测试平台，它集成了Chaos Mesh 和 Argo, 这是一个为 Kubernetes 设计的工作流引擎。 TiPocket设计成一个完全自动化的TiDB工程测试循环，这是一个分布式的数据库。 我们在进行混乱试验时采取了若干步骤，包括部署应用程序、工作量繁重、注射式例外和商业检查。 为了使这些步骤完全自动化，Argo被并入TiPocket。 Chaos Mesh提供丰富的故障注射，而Argo提供灵活的调节和调度安排。

![蒂口克特](/img/tipocket.png)



## 社区：从上到上建设

Chaos Mesh是一个由社区推动的项目，没有一个积极、友好和开放的社区，就不可能取得进展。 Chaos Mesh自开放源码以来，很快成为混乱工程领域中最具眼光的开放源码项目之一。 一年内，它在GitHub 和70多个贡献者上积累了3k星。 收养者包括Tencent Cloud、XPeng汽车、Dailymotion、NetEase Fuxi Lab、JuiceFS、APISIX和Meituan。 回顾过去一年，Chaos Mesh社区是从零建成的， 并为建立一个透明、开放、友好和自主的开放源码社区奠定了基础。



### 成为CNCF大家庭的一部分

云端人从一开始就在Chaos Mesh的DNA中。 加入该委员会是一项自然选择，这标志着Chaos Mesh成为供应商中立、开放和透明的开放源码社区的关键一步。 除了融入云端本土生态系统之外，加入CNCF会给予Chaos Mesh：

- 更多的社区和项目暴露。 与其他项目以及诸如Kubernetes Meetup和KubeCon等云端本土社区活动的合作，为我们提供了与社区沟通的极好机会。 我们感到惊奇的是，社区制作的高质量内容在促进Chaos Mesh方面也发挥了积极和深远的作用。

- 一个更完整和更开放的社区框架。 该委员会为开放源码社区行动提供了一个相当成熟的框架。 在全国妇女理事会的指导下，我们建立了基本的社区框架，包括行为守则、贡献指南和路线图。 我们也在 CNCF Slack下创建了我们自己的频道，#project-chaos-mesh。



### 友好和支持性社区

开放源码社区的质量决定了我们的收养者和贡献者是否愿意长期坚持和参与社区。 在这方面，我们一直在努力：

- 不断丰富文档并优化其结构。 迄今为止，我们为不同的受众群体编写了一整套文件。 包含 [个用户指南](https://chaos-mesh.org/docs/user_guides/installation/) 和 [开发者指南](https://chaos-mesh.org/docs/development_guides/development_overview), [快速启动指南](https://chaos-mesh.org/docs/get_started/get_started_on_kind), [使用案例](https://chaos-mesh.org/docs/use_cases/multi_data_centers), 和 [贡献指南](https://github.com/chaos-mesh/chaos-mesh/blob/master/CONTRIBUTING.md) 每次发布都会不断更新。

- 与社区合作，发布博客帖子、教程，使用案例和混乱的工程做法。 到目前为止，我们已经制作了26篇与Chaos Mesh有关的文章。 其中包括 [个交互式教程](https://chaos-mesh.org/interactiveTutorial), 发表在O'Reilly的Katakoda网站上。 这些材料是对文件的重要补充。

- 重新设计和扩充社区会议、网络研讨会和聚会中生成的视频和教程。 评价和答复社区反馈和询问。



## 展望未来

Google最近的全局停电提醒我们系统可靠性的重要性，它突出了混乱工程的重要性。 NCF TOC主席Liz Rice分享了 [2021年观看的5种技术](https://twitter.com/CloudNativeFdn/status/1329863326428499971)和混乱工程列在列表的顶端。 我们大胆地预测，混乱工程不久将进入一个新阶段。 1. Chaos Mesh 正处于积极发展阶段， 它包括社区要求，如嵌入的工作流程引擎，以支持更灵活的混乱情景的定义和管理， 应用状态检查机制和更详细的实验报告。  跟随项目 [路径图](https://github.com/chaos-mesh/chaos-mesh/blob/master/ROADMAP.md)。



## 最后但不是最小值

在过去一年中，Chaos Mesh的发展如此之大，但它仍然年轻，我们刚刚开始实现我们的目标。 同时，我们呼吁你们大家一起参与并帮助建造Chaos工程系统生态学！

如果您对Chaos Mesh感兴趣并想帮助我们改进它， 欢迎您加入 [我们的 Slack 频道](https://slack.cncf.io/) 或向我们 [GitHub 仓库提交您的合并请求或问题](https://github.com/chaos-mesh/chaos-mesh)。
