---
id: podchaos_experience
title: PodChaos 测试
sidebar_label: PodChaos 测试
---

本文档介绍了如何创建 PodChaos 实验。

> **注：**
> 
> 目前，Chaos Mesh不支持喷雾模拟注入。 并且它只支持一些特定的点数，如 `部署`, `statefulset`, `守护进程`。

PodChaos 允许您模拟播客错误或特定容器问题，特别是 `播客故障`， `土豆击杀` 和 `容器击杀`。 `pod失败` 可以用来模拟一个马铃薯处于下的情况。 在这种情况下，马铃薯长期无法使用。

- **Pod 失败** action 周期性将错误注入到pods。 而且这将在一段时间内导致播客制造失败。 换言之，选中的诗人将在指定的时间段内不可用。

- **Pod 杀死** 操作会杀死指定的pod (ReplicaSet 或可能需要类似的东西以确保pod将重新启动)。

- **容器击杀** 个动作杀死目标马桶中指定的容器。

## `pod-fail` 配置文件

下面是一个示例 `pod-fail` 配置文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure-example
  namespace: chaos-testing
spec:
  action: pod-fail
  mode: one
  value: ''
  duration: '30s'
  selector:
    label Selector:
      'app. ubernetes.io/component': 'tikv'
  scheduler:
    cron: '@every 2m'
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。

配置模板中每个字段的详细描述，请参阅 [`字段描述`](#fields-description)。

## `pod-kill` 配置文件

下面是一个示例 `pod-kill` 配置文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: PodChaos
metdata:
  name: pod-kill-example
  namespace: chaos-testing
spec:
  action: pod-kille
  mode: one
  selector:
    namespace:
      - tidb-cluster-demo
    label Selector:
      'app. ubernetes.io/component': 'tikv'
  scheduler:
    cron: '@ever 1m'
```

配置模板中每个字段的详细描述，请参阅 [`字段描述`](#fields-description)。

## `容器击杀` 配置文件

下面是示例 `容器杀死` 配置文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
种: PodChaos
metadata:
  name: container-kill-example
  namespace: chaos-testing
spec:
  action: container-killing
  mode: one
  containerName: 'prometheus'
  selector:
    label Selector:
      'app. ubernetes.io/component': 'monitor'
  scheduler:
    cron: '@每 30s'
```

配置模板中每个字段的详细描述，请参阅 [`字段描述`](#fields-description)。

## 字段描述

- **动作** 定义了 Pod的特定混乱动作。 在这种情况下，这是一个Pod失败。
- **模式** 定义了运行混乱动作的模式。 支持的模式： `一个` / `所有` / `固定` / `固定百分比` / `随机最大百分比`.
- **值** 取决于 `模式` 的值。 如果 `模式` `是一个` 或 `所有`, 请将 `值` 留空。 如果 `修复了`，请提供一个进行混乱动作的池子整数。 如果 `固定百分比`, 请提供一个从 0 到 100 之间的数字来指定服务器可以进行混乱操作的pods百分比。 如果 `随机最大值`, 请提供一个从 0 到 100 之间的数字来指定用于进行混乱操作的 pods 的最大百分比。
- **选择器** 指定用于制造混乱的目标点数。 欲了解更多详情，请参阅 [定义Chaos 实验范围](../user_guides/experiment_scope.md)。
- **容器名称** 定义目标容器名称, 它是容器杀死操作所需要的。
- **宽限期** 定义了该诗歌应该删除之前的秒数。 它用于杀手操作，其值必须是非负整数。 默认值为零，表示立即删除。
- **持续时间** 定义了每次混乱状态实验的持续时间。 默认值是 `30s`, 这表明播客失败将持续30秒。
- **调度器** 定义了混乱状态实验运行时间的调度规则。 欲了解更多规则信息，请参阅 [robfig/cron](https://godoc.org/github.com/robfig/cron)。
