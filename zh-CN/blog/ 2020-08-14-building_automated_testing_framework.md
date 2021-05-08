---
slug: /building_automated_testing_framework
title: 建立一个基于Chaos Mesh® and Argo 的自动测试框架
author: Ben Ye, Chengwen Yin
author_title: Chaos Mesh 维护者
author_url: https://github.com/chaos-mesh/chaos-mesh/blob/master/MAINTAINERS.md
author_image_url: https://avatars1.githubusercontent.com/u59082378?v=4
image: /img/automated_testing_framework.png
tags:
  - Chaos Mesh
  - Chaos工程
  - 测试自动化
---

![TiPocket - 自动测试框架](/img/automated_testing_framework.png)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)® 是 Kubernetes 开放源码工程平台。 尽管它提供了模拟异常系统条件的丰富能力，但它仍然只解决了Chaos Engineering 拼图中的一小部分。 除了故障注射外，完整的混乱工程应用还包括围绕确定的稳定状态进行假设。 运行生产实验，通过测试案例验证系统，并实现测试自动化。

这篇文章描述了我们如何使用 [TiPocket](https://github.com/pingcap/tipocket)， 一个自动测试框架，为TiDB构建一个完整的Chaos Engineering测试循环，我们已分发的数据库。

<!--truncate-->

## 我们为什么需要TiPocke？

在我们能够将像 [TiDB](https://github.com/pingcap/tidb) 这样的分布式系统投入生产之前，我们必须确保它足够健全，足以用于日常使用。 为此，几年前，我们将Chaos Engineering纳入了我们的测试框架。 在我们的测试框架中，我们：

1. 遵守正常的衡量标准并制定我们的测试假设。
2. 将失败列表插入TiDB。
3. 运行各种测试案例以验证错误场景中的 TiDB。
4. 监测和收集测试结果以供分析和诊断。

这听起来像一个稳固的过程，我们多年来一直使用它。 然而，随着TiDB的演变，试验比额表成倍增长。 我们有多种故障假设情景，在Kubernetes测试集群中运行数十个测试案例。 即使在Chaos Mesh帮助注入失败， 剩下的工作可能仍然很高——更不用说管道自动化以使测试能够扩大规模和效率的挑战。

这就是为什么我们建立了TiPocket，这是一个以Kubernetes和Chaos Mesh为基础的完全自动化测试框架。 目前，我们主要用它来测试TiDB组。 然而，由于TiPocket's Kubernetes友好设计和可扩展的接口，您可以使用 Kubernetes创建和删除逻辑，轻松支持其他应用。

## 如何工作

基于上述要求，我们需要自动工作流程：

- [注射破解](#injecting-chaos---chaos-mesh)
- [验证这种混乱的影响](#verifying-chaos-impacts-test-cases)
- [使混乱管道自动化](#automating-the-chaos-pipeline---argo)
- [可视化结果](#visualizing-the-results-loki)

### 注射破片段-Chaos Mesh

故障注入是核心混乱测试。 在分布式数据库中，任何时候都可能发生错误——从节点崩溃、网络分区和文件系统故障到内核恐慌。 Chaos Mesh正是在这里出现的。

目前，TiPocket 支持以下类型的故障注入：

- **网络**: 模拟网络分区、 随机数据包丢失、损坏、重复或链接延迟。
- **时间扭曲**: 模拟要测试的容器的时钟扭曲。
- **杀死**: 在集群或组件内随机杀死指定的土豆(TiDB, TiKV, 或放置驱动程序(PD))。
- **I/O**: 注入I/O 延迟TiDB的存储引擎，TiKV，以识别与I/O 相关的问题。

在处理错误注入时，我们需要思考验证问题。 我们如何确保TiDB能够生存这些缺陷？

## 验证混乱的影响：测试案件

为了证实TiDB如何经得起混乱，我们在TiPocket执行了几十个试验案例，并配备了各种检查工具。 为了让您了解TiPocket如何验证失败时的TiDB，请考虑以下测试案例。 这些案件的重点是SQL的执行、交易一致和交易孤立。

### Fuzz 测试：SQLsmith

[SQLsmith](https://github.com/pingcap/tipocket/tree/master/pkg/go-sqlsmith) 是一个生成随机SQL 查询的工具。 TiPocket 创建 TiDB 集群和 MySQL 实例... SQLsmith 生成的随机SQL 在 TiDB 和 MySQL 上执行，并且将各种错误注入TiDB 集群进行测试。 最后，对执行结果进行比较。 如果我们发现不一致之处，我们的系统就会有潜在问题。

### 交易一致性测试：银行和波库文

[Bank](https://github.com/pingcap/tipocket/tree/master/cmd/bank) 是一个典型的测试案例, 它模拟了银行系统中的转账过程。 在短暂孤立的情况下，所有转账必须确保所有账户的总金额在任何时候都必须一致。 即使面对系统故障。 如果总金额有不一致之处，我们的系统就可能存在问题。

[Portcupine](https://github.com/anishathalye/porcupine) 是一个去建造的线性检查器，用于测试分布式系统的正确性。 它需要一个作为可执行的 Go 代码的顺序规格以及一个并行的历史。 它决定历史是否可以线性地说明顺序。 在TiPocket，我们在多个测试案例中使用 [Porcupine](https://github.com/pingcap/tipocket/tree/master/pkg/check/porcupine) 检查器来检查TiDB 是否符合线性限制。

### 交易隔离测试：Elle

[Elle](https://github.com/jepsen-io/elle) 是一个检查工具，用于验证数据库的交易隔离等级。 TiPocket 整合了 [go-elle](https://github.com/pingcap/tipocket/tree/master/pkg/elle)，Elle 检查工具的实现，以验证TiDB的隔离水平。

TiPocket用于验证TiDB的准确性和稳定性。 欲了解更多测试案例和验证方法，请参阅我们的 [源代码](https://github.com/pingcap/tipocket)。

## 管道自动化 - Argo

现在我们有 Chaos Mesh来注入错误，一个 TiDB 集群来测试。 和验证TiDB的方法。我们如何实现混乱测试管道的自动化？ 我们想到两个选项：我们可以实现TiPocket的日程安排功能，或者将工作移交给现有的开源工具。 为了使TiPocket更专注于测试我们的工作流，我们选择了开源工具方法。 这加上我们所有K8s设计，直接引导我们到 [Argo](https://github.com/argoproj/argo)。

Argo是一个为Kubernetes设计的工作流引擎。 长期以来它一直是一种开放源码产品，并得到广泛的关注和应用。

Argo抽象了一些自定义资源定义（CRDs）用于工作流。 其中最重要的包括工作流程模板、工作流程和Cron 工作流程。 以下是 Argo 如何适合TiPocket：

- **工作流模板** 是为每个测试任务预先定义的模板。 测试运行时可以传递参数。
- **工作流** 以不同的顺序安排多个工作流模板，这些模板构成要执行的任务。 Argo还可以让您在管道中添加条件、循环和定向环形图 (DAGs)
- **Cron Workflow** 允许您安排一个工作流类似于cron 作业的工作流。 它完全适合你想要长期执行测试任务的场景。

我们预定义银行测试的样本工作流程显示如下：

```yml
spec:
  entrypoint: call-tipocket-bank
  参数:
    参数:
      - 名称: ns
        值: tipocket-bank
            - 名称: nemesis
        值: random_kill, ill_pd_leader_5min,partition_one,subcritical_skews,big_skews,shuffle-leader-scheduler,shuffle-region-scheduler, 随机合并调度程序
  模板：
    - 名称：call-tipocket-bank
      步骤：
        - 名称：call-wait-cluster
            模板：
              名称：等候集群
              模板：等待集群
        - 名称：call-tipocket-bank
            模板：
              name: tipocket-bank
              模板：tipocket-bank：tipocket-bank。
```

在此示例中，我们使用 Workflow 模板和 nemesis 参数来定义注入的特定失败。 您可以重新使用模板来定义适合不同测试情况的多个工作流。 这允许您在流程中添加更多自定义的故障注入。

除了 [TiPocket的](https://github.com/pingcap/tipocket/tree/master/argo/workflow) 示例工作流程和模板，设计还允许您添加自己的失败注入流程。 使用可编解码的工作流处理复杂的逻辑，使阿尔戈对开发者友善，成为我们场景的理想选择。

现在，我们的混乱实验正在自动运行。 但如果我们的结果不能满足我们的期望？ 我们如何找到问题？ TiDB保存各种监测信息，这使得日志收集对TiPocket的观测能力至关重要。

## 可视化结果：Loki

在云层系统中，观察能力非常重要。 一般而言，您可以通过 **度量**、 **日志**和 **追踪** 实现观测。 TiPocket的主要测试案例用于评估 TiDB 集群，所以计数和日志是我们查找问题的默认来源。

关于Kubernetes，Prometheus 是衡量尺度的实际标准。 但是，没有通用的方式来收集日志。 解决方案如 [Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch), [Fluent Bit](https://fluentbit.io/), 和 [Kibana](https://www.elastic.co/kibana) 运行良好，但可能引起系统资源争执和维护费用高昂。 我们决定使用 [Loki](https://github.com/grafana/loki), 来自 [Grafana](https://grafana.com/) 的 Prometheus 类的日志聚合系统。

Prometheus 正在处理 TiDB 的监测信息。 Prometheus和Loki有类似的标签系统。 这样我们就可以轻松地将Promethus的监测指标与相应的诗歌日志结合起来，并使用类似的查询语言。 Grafana还支持Loki仪表板，这意味着我们可以使用Grafana同时显示监测指标和日志。 Grafana 是TiDB的内置监测组件，Loki可以重新使用。

## 将它们放在一起-TiPocket

现在，一切都已就绪。 以下是TiPocket的简化图表：

![TiPocket 架构](/img/tipocket-architecture.png)

正如你可以看到的那样，Argo Workflow 管理所有混乱的实验和测试案例。 一般而言，一个完整的试验周期涉及下列步骤：

1. Argo创建了一个 Cron 工作流程，它定义了需要测试的集群、注入的错误、测试案例和任务期限。 如有必要，Cron 工作流也允许您实时查看案例日志。

![Argo 工作流](/img/argo-workflow.png)

1. 在指定的时间里，工作流中开始了一个单独的TiPocket 线程，并启动了Cron 工作流。 TiPocket发送TiDB-Operator来测试集群的定义。 而TiDB-Operator 则创建一个目标TiDB集群。 同时，Loki收集相关的日志。
2. Chaos Mesh在集群中注入故障。
3. 用户使用上述测试案例验证系统的健康状况。 任何测试失败都会导致Argo的工作流故障，导致AlertManager将结果发送到指定的 Slack 通道。 如果测试案例正常完成，集群将被清理，阿戈将站在下一次测试之前。

![Slack 中的警报](/img/alert_message.png)

这是完整的 TiPocket 工作流。 .

## 加入我们

[Chaos Mesh](https://github.com/pingcap/chaos-mesh) and [TiPocket](https://github.com/pingcap/tipocket) 都在活动迭代。 我们捐赠了Chaos Mesh给 [CNCF](https://github.com/cncf/toc/pull/367)， 我们期待着更多的社区成员与我们一道建立一个完整的Chaos Engineering生态系统。 如果这听起来对您很感兴趣，请查看我们的 [网站](https://chaos-mesh.org/)，或加入 #chaos-mesh在 [Slack](hthttps://slack.cncf.io/) 中。
