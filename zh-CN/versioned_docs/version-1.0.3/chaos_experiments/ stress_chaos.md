---
id: straos_experience
title: 压力沙托斯实验
sidebar_label: 压力沙托斯实验
---

本文档有助于您建立压力链实验。

压力链能够对收集的药水产生大量压力。 压力器通过内部 `chaos-daemon` 注入到目标pods。

## 配置

RestricsChaos像其他混乱一样具有共同的配置，如如何选择诗人，如何确定周期性的混乱。 您可以参考其他文档了解详情。 它在 **中定义了下面两个路径的** 个压力器：

- `应力器`

  `压力器` 定义了大量支撑压力系统部件的压力器。 你可以使用其中的一个或多个来弥补各种压力。 至少应指定其中一个应激反应器。 以下是当前支持的压力器：

  1. `内存`

     `内存` 压力器将持续压低虚拟内存。

     | 选项       | 类型 | 必填 | 描述        |
     | -------- | -- | -- | --------- |
     | `B. 工 人` | 整数 | 真的 | 指定并行评分实例。 |

  2. `奇普文`

     `cpu` 压力器将持续将 CPU 压出.

     | 选项       | 类型 | 必填 | 描述                                         |
     | -------- | -- | -- | ------------------------------------------ |
     | `B. 工 人` | 整数 | 真的 | 指定并行评分实例。 实际上它指定了在它少于可用的 CPU 时需要强调的CPU 数量。 |
     | `负载`     | 整数 | 错误 | 指定每个工人加载百分比数。 0 实际上是一个睡眠(无负载)，100个正在满载.    |

- `压力压力计压器`

  `压力强度` 定义了大量的压力强度，就像 `压力强度` 只是试验性特征和更强大。

  您可以在 `stres-ng` 中定义压力器(另见 `manstres-ng`) 方言。

  > **注：**
  > 
  > 然而，并非所有受支持的压力都经过良好的测试。 它可能会在以后的版本中退出。 因此， 建议使用 `压力器` 来定义压力器，并且只有在您想要更多压力器得不到 `压力器支持时才使用压力器`

  如果同时界定了 `个压力调度器` and `压力调度器` 则 `压力调度器` 温度调度器。

## 用法

下面是一个 YAML RestesChaos 示例文件，它将在每2分钟内燃烧一个 CPU 30秒：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: Restreschaos
metdata:
  name: burn-cpu
  namespace: chaos-testing
spec:
  mode: one
  selector:
    namespace:
      - tidb-cluster-demo
  strators:
    cpu:
      workers: 1
  dur: '30s'
  scheduler:
    cron: '@every 2m'
```

1. 为您的应用程序创建一个命名空间。 例如， `tidb-cluster-demo`:

   ```bash
   kubectl 创建 ns tidb-cluster-demo
   ```

2. 在目标命名空间中创建您的pods：

   ```bash
   kubectl apply -f *your pods.yaml*
   ```

3. 注入压力链：

   ```bash
   kubectl apply -f *your stress-chaos.yaml*
   ```

然后，你的Pod 的 CPU 将燃烧30秒。
