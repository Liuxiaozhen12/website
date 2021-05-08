---
id: timechaos_experience
title: TimeChaos 测试
sidebar_label: TimeChaos 测试
---

本文件描述了如何在Chaos Mesh中添加TimeChaos实验。

时间Chaos用于修改 `时钟时间`的返回值，这会导致Go `时间偏移。 ow()` and Rust std's `std::time::Instant::now.()` etc.

## 配置文件

下面是一个 TimeChaos 配置文件样本：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: TimeChaos
metadata:
  name: timeshift-example
  namespace: chaos-testing
spec:
  mode: one
  selector:
    labeleselector:
      'app. 伯尔尼。 o/component': 'pd'
  timeOffset: '10m100ns'
  clockIds:
    - CLOCK_REALTIME
  containername:
    - pd
  duration: '10s'
  scheduler:
    cron: '@每 15'
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。

描述：

- **模式** 定义了选择pods的模式。
- **选择器** 指定了用于故障注入的目标点。 欲了解更多详情，请参阅 [定义Chaos 实验范围](../user_guides/experiment_scope.md)。
- **时间偏移** 指定时间偏移。 这是一个指定单位的持续时间字符串, 例如 `300ms`, `-1.5h`. 有效时间单位是“ns”、“us”(或“微秒”)、“ms”、“s”、“m”、“h”。
- **时钟ID** 定义了所有受影响的 `clk_id`. `clk_id` 指的是 `时钟时间` 调用的第一个参数。 对于大多数应用， `CLOCK_REALTIME` 就足够了。
- **容器名称** 选择受影响的容器名称。 如果未设置，所有容器将被注入。
- **持续时间** 定义了每次混乱状态实验的持续时间。 在上面的样本文件中，时间混乱持续10秒。
- **调度器** 定义了混乱状态实验运行时间的调度规则。 欲了解更多规则信息，请参阅 [robfig/cron](https://godoc.org/github.com/robfig/cron)。

## 限制

- 时间修改只能注入主容器过程。
- 时间故障对纯粹的系统调用 `时钟时间` 没有影响。
- 所有已注入的 [vDSO](http://man7.org/linux/man-pages/man7/vdso.7.html) 调用都使用纯系统调用来获得实时，因此与时钟相关的函数调用可能要慢得多。
