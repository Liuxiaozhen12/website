---
slug: /如何进行自上而下的游戏公司-使用-chaos-engineering-to improv-standing
title: '顶级游戏公司如何使用Chaos Engineering来改进测试'
author: Hui Zhang @ Fuxi Lab, NetEase
image: /img/fuxi-cash banner.jpg
tags:
  - Chaos Mesh
  - Chaos工程
---

![How-a-Top-Game-Company-Uses-Chaos-Engineering以改进测试](/img/fuxi-case-banner.jpg)

NetEase Fuxi AI 实验室是第一家专业游戏AI研究机构。 研究人员使用我们以Kubernetes为基础的Danlu平台来开发算法，进行培训和调整，以及在线出版。 由于与Kubernetes的整合，我们的平台效率要高得多。 然而，由于与Kubernetes和微型服务有关的问题，我们正在不断测试和改进我们的平台，使其更加稳定。

<!--truncate-->

在这篇文章中，我将讨论我们最宝贵的测试工具之一， [Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)。 Chaos Mesh是一个开放源码工程工具，通过其仪表板提供广泛的故障注入和极好的故障监测。

## 为什么Chaos Mesh

我们在2018年开始寻找Chaos Engineering工具。 我们正在寻找一个工具：

- 云生支持。 Kubernetes实际上是服务调节和安排的实际标准，应用程序运行时间已经完全标准化。 对于完全在 K8s上运行的应用程序来说，云端本地支持是与它们相关的任何工具所必须的。

- 足够的故障注入类型。 对状态服务而言，网络故障模拟特别重要。 平台必须能够模拟不同级别的失败，例如Pods、网络和I/O。

- 良好的观察能力。 知道一个故障何时注入以及何时能够恢复，对于我们判断应用程序中是否存在异常现象至关重要。

- 社区积极支持。 我们想要使用一个经过彻底测试和始终如一的开放源码项目。 这就是为什么我们珍视持续和及时的社区支持。

- 无需域知识而进入现有应用程序。

- 实际使用案例供我们评估和发扬光大。

2019年，当Chaos Mesh为Kubernetes设计的Chaos Engineering 平台开放源码时，我们发现了我们正在寻找的工具。 它仍然处于早期阶段；然而，我们立即受到它所支持的大量过失类型的打击。 这比其他混乱工程工具具有很大的优势，因为在某种程度上， 它决定了我们能够在系统中找到的问题数量。 我们立即意识到，Chaos Mesh几乎完全达到了我们的期望。

![Chaos Mesh 架构](/img/chaos-mesh-architecture.png)

## 我们与Chaos Mesh 的旅程

Chaos Mesh帮助我们找到了几个重要的漏洞。 例如，它在 [rabbitMQ](https://www.rabbitmq.com/)中检测到一个大脑分割的问题，这是丹麦语的开源消息队列软件。 根据 [Wikipedia](https://en.wikipedia.org/wiki/Split-brain), “分裂大脑状态指因维持两套单独的数据集而造成的数据或可用性不一致，在范围上有重叠。” 当一个rabbitMQ集群出现大脑分离错误时，将会出现数据写冲突或错误， • 造成更严重的问题，如信息传递服务中数据不一致的问题。 正如我们在下面的架构中所显示的那样，当人才分裂发生时，消费者不会正常运作，也不会保留服务器异常。

![RabbitMQ集群的结构](/img/architecture-of-a-rabbitmq-cluster.png)

通过Chaos Mesh，我们可以将 `土豆杀死` 个错误注入我们的容器实例云，从而稳固地复制这个问题。

Chaos Mesh还发现其他一些问题，包括启动失败， 一个联机故障经纪集群、心超时和连接通道关机。 随着时间的推移，我们的发展小组解决了这些问题，并大大改善了丹卢平台的稳定。

## 一个快速成长的项目

Chaos Mesh不断更新和改进。 当我们首次通过它时，它甚至没有达到稳定的版本。 它没有调试或日志收集工具，控制面板组件仅适用于TiDB。 我们可以使用Chaos Mesh测试其他应用程序的唯一方法是通过 `kubectl 应用程序执行 YAML 配置文件`。

[Chaos Mesh 1.0](https://chaos-mesh.org/blog/chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-easier) 已修复或改进了大多数这些限制。 它提供了更加精细和强大的混乱支持，普遍提供了Chaos Dashboard，增强了观测能力，并提供了更准确的混乱范围控制。 所有这些都是由一个开放、协作和充满活力的社区驱动的。

![Chaos 仪表盘现在一般可用](/img/chaos-dashboard.gif)

## 展望未来

看到Chaos Mesh 已经成长了多少以及它的收获多少，这是令人惊奇的。 我们也对我们在这方面取得的成就感到高兴。

然而，Chaos Engineering是一个需要开展工作的大领域。 今后，我们希望看到以下功能：

- 原子故障输入

- 将自定义的故障类型与验证实验对象的标准化方法结合起来的未接生故障

- 一般组件如MySQL、Redis和Kafka的标准测试案件

我们已经与那些维护Chaos Mesh的人讨论了这些功能，他们说这些功能已经在Chaos Mesh 2.0 路径图上。

如果您感兴趣，通过 [Slack](https://slack.cncf.io/) (#project-chaos-mesh)或 [GitHub](https://github.com/chaos-mesh/chaos-mesh) 加入Chaos Mesh社区。
