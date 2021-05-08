---
id: networkchaos_experience
title: NetworkChaos 测试
sidebar_label: NetworkChaos 测试
---

本文档描述了如何在Chaos Mesh中创建NetworkChaos实验。

NetworkChaos行动分为两类：

- **网络分区** 动作通过阻止pods 之间的通信将pods 分割成几个独立的子网。

- **网络仿真(Netem) Chaos** 动作涵盖网络的正常故障，如网络延迟、重复、丢失和腐败。

## 网络分区动作

下面是一个示例网络分区配置文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
种: NetworkChaos
metadata:
  name: network-partition-example
  namespace: chaos-testing
spec:
  action: 
 action: 分区
  模式: one
  selector:
    label Selector:
      'app. 伯尔尼。 o/component': 'tikv'
  direction: to
  target:
    selector:
      namespaces:
        - tidb-cluster-demo
      label Selector:
        'app. 伯尔尼。 o/component: 'tikv'
    模式: one
  duration: '10s'
  scheduler:
    cron: '@每 15'
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。

描述：

- **操作** 定义了针对pod的特定混乱动作。 在此情况下，它是网络分区。
- **模式** 定义了运行混乱动作的模式。
- **选择器** 指定了用于故障注入的目标点。 欲了解更多详情，请参阅 [定义Chaos 实验范围](../user_guides/experiment_scope.md)。
- **方向** 指定了分区方向。 支持的方向是 `从`, `到` 和 `都是`
- **目标** 指定网络混乱动作的目标。
- **外部目标** 指定 Kubernetes 集群之外用于网络混乱动作的目标。
- **持续时间** 定义了每次混乱状态实验的持续时间。 在上面的样本文件中，网络分区持续 `10` 秒。
- **调度器** 定义了混乱状态实验运行时间的调度规则。 欲了解更多规则信息，请参阅 [robfig/cron](https://godoc.org/github.com/robfig/cron)。

## Netem Chaos 操作

有四起失踪案，即丢失、延误、重复和腐败行为。

> **注：**
> 
> 配置模板中每个字段的详细描述与 [网络分区](#network-partition-action) 中的描述一致。

### 网络丢失

网络丢失动作导致网络数据包随机丢弃。 若要添加网络丢失动作，请在 [`/examps`](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/network-loss-example.yaml) 中找到并编辑相应的模板。

在这种情况下，需要两种特定动作属性——丢失和相关性。

```yaml
损失：
  损失：'25'
  关联：'25'
```

**丢失** 定义了丢包的百分比。

NetworkChaos变量并非纯属随机性，因此可以模拟也有相关值。

### 网络延迟

网络延迟操作会导致消息发送延迟。 若要添加网络延迟动作，请在 [`/examps`](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/network-delay-example.yaml) 中找到并编辑相应的模板。

在这种情况下，需要三个特定的动作属性——相关性、干扰性和延迟性。

```yaml
延迟:
  延迟: '90毫秒'
  correlation: '25'
  jitter: '90毫秒'
```

**延迟** 定义了发送数据包的延迟时间。

**Jitter** 指定了延迟时间的垃圾。 默认值是 `0ms`。 Jitter 在技术上也被称为数据包延迟变化。

**关联** 指定了喷射器的关联性。 默认值是 `0`。

在上述示例中，网络延迟为90毫安±90毫秒，25%的相关性。

为了更好地了解Jitter、延迟和网络延迟，你可以阅读 [这篇文章](https://www.speedcheck.org/wiki/jitter/)。

### 网络重复

网络重复操作导致数据包重复。 若要添加网络重复操作，请在 [`/examps`](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/network-duplicate-example.yaml) 中定位和编辑相应的模板。

在这种情况下，需要两个属性――相关性和重复。

```yaml
重复:
  重复: '40'
  相关: '25'
```

**重复** 表示数据包重复的百分比。 在上述例子中，重复率是40%。

### 网络损坏

网络腐败行为导致数据包损坏。 若要添加网络损坏行为，请在 [`/examps`](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/network-corrupt-example.yaml) 中定位和编辑相应的模板。

在这种情况下，需要两个特定动作的属性——相关性和损坏。

```yaml
损坏:
  损坏: '40'
  关联: '25'
```

**已损坏** 指定数据包损坏的百分比。

## 网络带宽动作

网络带宽操作用于限制网络带宽。 若要添加网络带宽动作，请在 [`/examps`](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/network-bandwidth-example.yaml) 中找到并编辑相应的模板。

> **注：**
> 
> Minikube不支持此功能，因为 `Minikube的图像已禁用了 CONFIG_NET_SCH_TBF`。

若要注入网络带宽错误，需要三个特定动作属性——速率、缓冲区和限制。

```yaml
带宽：
  速率：10 kbps
  缓冲：1000
  限制 ︰ 100
```

**比率** 允许"bps", "kbps", "mbps", "gbps", "tbps". “bps”是指每秒字节。

**限制** 定义可以等待令牌可用队列的字节数。

**缓冲区** 是令牌可以即时使用的最大字节数。

**峰值** 是桶的最大耗减率。

**minburst** 指定了峰率桶的大小。
