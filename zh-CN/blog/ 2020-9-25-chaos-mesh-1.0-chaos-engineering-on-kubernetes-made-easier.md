---
slug: /chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-made-feed
title: 'Chaos Mesh 1.0: Kubernetes Mae Easier 上的Chaos Engineering'
author: Chaos Mesh 维护者
author_url: https://github.com/chaos-mesh
author_image_url: https://avatars1.githubusercontent.com/u59082378?v=4
image: /img/chaos-mesh-1.0.png
tags:
  - 通 知
  - Chaos Mesh
  - Chaos工程
---

![Chaos-Mesh-1.0 - Chaos-Engineering-on-Kubernetes-Made-Easier](/img/chaos-mesh-1.0.png)

今天，我们自豪地宣布Chaos Mesh® 1。 , 在2020年7月作为 [沙盒项目](https://pingcap.com/blog/announcing-chaos-mesh-as-a-cncf-sandbox-project) 加入国家合作框架。

<!--truncate-->

Chaos Mesh 1.0 是该项目发展的一个重要里程碑。 经过10个月的开放源码社区的努力，Chaos Mesh在功能、可扩展性和易用性方面已经准备就绪。 这里有一些重点。

## 强大的混乱支持

[Chaos Mesh](https://chaos-mesh.org) 源自 [TiDB](https://pingcap.com/products/tidb)的测试框架， a 分布式数据库，因此考虑到分布式系统可能存在的缺陷。 Chaos Mesh提供了全面和精细的故障类型，包括Pod、网络、系统I/O和内核。 在YAML中定义了Chaos实验，它很快且易于使用。

Chaos Mesh 1.0 支持以下故障类型：

- 时钟偏差：模拟时钟偏差
- 容器击杀：模拟正在被杀死的容器
- cpu-burn: 模拟 CPU 压
- io-attribtion-override: 模拟文件异常
- io-default: 模拟文件系统I/O 错误
- io-延迟：模拟文件系统I/O 延迟
- 内核注入：模拟内核失败
- 内存燃烧：模拟内存压力。
- 网络损坏：模拟网络数据包损坏
- 网络重复：模拟网络数据包重复
- 网络延迟：模拟网络延迟
- 网络损失：模拟网络损失
- 网络分区：模拟网络分区
- pod-failment: 模拟Kubernetes Pods 持续不可用
- pod-kill: 模拟正在被击杀的 Kubernetes Pod

## 视觉故障管道化

Chaos 仪表板组件是一个一站式网络接口，供Chaos Mesh用户管理混乱试验。 以前，Chaos 仪表板只能测试TiDB。 使用 Chaos Mesh 1.0，人人都可以使用。 Chaos 仪表板极大地简化了混乱试验的复杂性。 只点击几个鼠标，您可以定义chaos试验的范围，指定Chaos注入类型， 定义调度规则并观察混乱状态试验的结果——所有这些都在同一个网络接口中。

![Chaos 仪表板](/img/chaos-dashboard.gif)

## Grafana 插件用于增强观测能力

为了进一步提高混乱试验的可观察性，Chaos Mesh 1。 包括一个 Grafana 插件，以允许您在应用程序监测面板上直接显示实时混乱的实验信息。 目前，混乱状态实验信息被显示为注释。 这样，您可以同时观察应用程序的运行状态和当前的混乱状态实验信息。

![Grafana 上的Chaos 状态和应用程序状态](/img/chaos-status.png)

## 安全和可控制的故障

当我们进行混乱试验时，我们必须严格控制混乱的范围或“爆炸性辐射”。 1. Chaos Mesh 不仅提供了大量的选择者来准确地控制试验的范围， 但它也能让您设置受保护的命名空间来保护重要的应用程序。 您也可以使用命名空间权限将Chaos Mesh的范围限制为特定的命名空间。 这些特征加在一起，使Chaos Mesh的混乱试验变得安全和可控制。

## 现在试试

您可以通过 `install.sh` 脚本或Helm 工具在您的 Kubernetes 环境中快速部署Chaos Mesh。 关于具体安装步骤，请参阅 [Chaos Mesh 入门](https://chaos-mesh.org/docs/user_guides/installation) 文档。 此外，由于 [Katakoda交互式教程](https://chaos-mesh.org/interactiveTutorial)， 你也可以很快获得你在Chaos Mesh的手，不必部署它。

如果您尚未升级到 1.0 GA ，请参阅 [1.0 发布说明](https://github.com/chaos-mesh/chaos-mesh/releases/tag/v1.0.0) 以了解更改和升级指南。

## 谢谢！

感谢我们所有的Chaos Mesh [贡献者](https://github.com/chaos-mesh/chaos-mesh/graphs/contributors)！

如果您对Chaos Mesh感兴趣，欢迎您通过提交问题或贡献代码、文档或文章加入我们。 我们期待你们的参与和反馈！
