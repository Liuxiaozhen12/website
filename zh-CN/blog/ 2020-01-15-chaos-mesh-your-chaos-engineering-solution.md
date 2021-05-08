---
slug: /chaos_mesh_your_chaos_engineering_solution
title: Chaos Mesh - Kubernetes系统复原力的Chaos Engineering Solution
author: Cwen Yin
author_title: Chaos Mesh 维护者
author_url: https://github.com/cwen0
author_image_url: https://avatars1.githubusercontent.com/u/22956341?v=4
image: /img/chaos-engineering.png
tags:
  - Chaos Mesh
  - Chaos工程
  - Kubernetes
---

![Chaos工程](/img/chaos-engineering.png)

## 为什么Chaos Mesh？

在分布式计算机的世界中，您的集群在任何时候、任何地方都可能发生错误。 传统上，我们有单位测试和集成测试，以保证一个系统已经准备就绪。 但这些仅仅涵盖冰山一角的群组规模、复杂性和数据量增加PB水平。 更好地查明系统的脆弱性并提高复原力， Netflix发明了 [Chaos Monkey](https://netflix.github.io/chaosmonkey/) 并将各种错误注入基础设施和商业系统。 这就是Chaos Engineering的发源方式。

<!--truncate-->

在 [PingCAP](https://chaos-mesh.org/), 我们在创建 [TiDB](https://github.com/pingcap/tidb)时面临同样的问题，这是一个开放源分发的 NewSQL 数据库。 要成为过错容忍者或复原力强的人，对我们来说尤其如此。 因为这关系到任何数据库用户最重要的资产，即数据本身。 为了确保复原力，我们从很早阶段就在我们的测试框架内开始了 [练习Chaos Engineering](https://pingcap.com/blog/chaos-practice-in-tidb/)。 然而，随着TiDB的增加，测试要求也随之增加。 我们认识到，我们需要一个普遍的混乱测试平台，不仅对TiDB，而且对其他分布式系统也是如此。

因此，我们向你们展示Chaos Mesh是一个云层土生土长的Chaos Engineering平台，它在Kubernetes环境中引发混乱试验。 它是一个开放源代码项目，可在 [https://github.com/chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh) 获取。

在以下章节中，我将与你们分享Chaos Mesh是什么，我们如何设计和执行它。 最后我会告诉你你如何在你的环境中使用它。

## Chaos Mesh 能做什么？

Chaos Mesh是一个多功能Chaos Engineering 平台，其特点是Kubernetes复杂系统的全环故障注射方法。 覆盖Pod、网络、文件系统，甚至内核中的错误。

这是我们如何使用 Chaos Mesh来定位一个 TiDB 系统缺陷的一个例子。 在这个示例中，我们用分布式存储引擎模拟Pod 故障时间([TiKV](https://pingcap.com/docs/stable/architecture/#tikv-server))并观察每秒查询的变化(QPS)。 如果一个TiKV节点被关闭，QPS可能会在返回到失败前的水平之前遇到一个短暂的喷雾器。 这就是我们如何保证高供应量的方式。

![Chaos Mesh 在 TiKV 中发现停时恢复异常](/img/chaos-mesh-discovers-downtime-recovery-exceptions-in-tikv.png)

<div class="caption-center"> Chaos Mesh 在 TiKV 中发现停时恢复异常 </div>

您可以从仪表板中看到：

- 在前两次下调期间，QPS在大约1分钟后恢复正常。
- 然而，在第三次停机后，检疫和装运前检索需要更长的时间——大约9分钟。 这么长的停机时间是意料不到的，肯定会影响到在线服务。

经过一些诊断，我们在测试后发现TiDB群组版本(V3.0.1)在处理TiKV时有一些棘手的问题。 我们在以后的版本中解决了这些问题。

但Chaos Mesh不仅仅是模拟停顿时间，还能做很多事情。 它还包括这些故障注入方法：

- **pod-kill:** 模拟正在杀死Kubernetes Pods
- **pod-failment:** 模拟Kubernetes Pods 持续不可用
- **网络延迟：** 模拟网络延迟
- **网络流失：** 模拟网络流失包
- **网络重复：** 模拟网络数据包
- **网络损坏：** 模拟网络数据包损坏
- **网络分区：** 模拟网络分区
- **I/O 延迟：** 模拟文件系统 I/O 延迟
- **I/O 错误:** 模拟文件系统I/O 错误

## 设计原则

我们为Kubernetes设计了Chaos Mesh易于使用、可缩放和设计。

### 简单易用

Chaos Mesh 要便于使用，必须：

- 不需要特殊依赖关系，这样它就可以直接部署到 Kubernetes 集群，包括 [Minikube](https://github.com/kubernetes/minikube)。
- 不需要修改试验系统(SUT)的部署逻辑，以便在生产环境中进行混乱试验。
- 轻松地调节故障注射在混乱实验中的行为，并轻松地查看实验状态和结果。 您也应该能够快速回滚注入失败。
- 隐藏潜在的实现细节，以便用户能够集中精力整理混乱试验。

### 可缩放

Chaos Mesh应该可以伸缩，以便我们可以方便地在不重启轮子的情况下将新的要求插入。 具体地说，Chaos Mesh必须：

- 利用现有的实现方法使故障注入方法能够轻松缩放。
- 与其他测试框架轻松结合。

### 为Kubernetes设计

在集装箱世界中，Kubernetes是绝对的领先者。 它的采纳率远远超出了每个人的期望，它赢得了集装箱化杂乱无章的战争。 从本质上说，Kubernetes是云层的操作系统。

TiDB 是一个云端分布式数据库。 我们内部的自动化测试平台从一开始就建立在Kubernetes上。 我们每天都有数百个TiDB集群在Kubernetes上运行，进行各种试验。 包括广泛的混乱测试，以模拟生产环境中所有类型的故障或问题。 为了支持这些混乱局面的试验，混乱局面与Kubernetes的结合成了我们实施这一计划的自然选择和原则。

## CustomResourceDefinition 设计

Chaos Mesh 使用 [CustomResourceDefinition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CRD) 定义chaos objects. 在Kubernetes领域，CRD是一个成熟的解决方案来执行自定义资源，有大量的执行案例和工具集。 使用 CRD 使Chaos Mesh自然融入Kubernetes 生态系统。

而不是在一个统一的 CRD 对象中定义所有类型的故障注入， 我们允许对不同类型的故障注入灵活和单独的 CRD 对象。 如果我们添加了一个符合现有CRD对象的故障注入方法，我们直接基于此对象缩放； 如果它是一个全新的方法，我们为此创建一个新的 CRD 对象。 通过这个设计，从顶层提取混乱对象定义和逻辑实现，从而使代码结构更加清晰。 这种做法还降低了合并的程度和错误的概率。 此外，Kubernete's [controller-runtime](https://github.com/kubernetes-sigs/controller-runtime) 是实现控制器的优秀包装器。 这为我们节省了很多时间，因为我们不必重复实现每个CRD项目的同一组控制器。

Chaos Mesh实现了PodChaos、NetworkChaos和IOChaos对象。 姓名明确标明相应的故障注入类型。

例如，Pod 粉碎是Kubernetes环境中一个非常常见的问题。 许多本地资源对象自动处理这种错误，比如创建一个新的 Pod。 但我们的应用能否真正处理这种错误？ 如果Pod 不会开始什么呢？

使用 `Pod-kill`等明确定义的操作，PodChaos 可以帮助我们更有效地确定这些类型的问题。 PodChaos 对象使用以下代码：

```yml
示例：
 操作：Pod-kill
 模式：一个
 选择器：
   命名空间：
     - tidb-cluster-demo
   labelSelector：
     "app.kubernetes.io/component"："tikv"
  scheduler：
   cron：“@every 2m”
```

此代码执行以下操作：

- `动作` 属性定义了要注入的特定错误类型。 在这种情况下， `杀死` 随机杀死pod。
- `选择器` 属性将chaos 实验的范围限制在一个特定的范围。 在这种情况下，TiDB 群集的范围是 `tidb-cluster-demo` 命名空间的TiKV podes。
- `调度器` 属性定义了每次故障动作的间隔。

关于 CRD 对象，如NetworkChaos 和 IOChaos 的详细信息，请参阅 [Chaos-mesh文档](https://github.com/chaos-mesh/chaos-mesh)。

## Chaos Mesh 是如何工作的？

CRD设计已经解决了，让我们看一看Chaos Mesh如何工作的大画面。 2. 涉及下列主要组成部分：

- **控制器管理器**

  作为平台的"大脑"。 它管理CRD对象的生命周期和计划混乱试验。 它有用于schedule CRD 对象实例的对象控制器， [admission-webhooks](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) controller 动态地将侧边际容器注入到 Pods。

- **chaos-daemon**

  运行为有特权的守护程序, 可以在节点和群组上运行网络设备。

- **sidecar**

  运行为特殊类型的容器，由接入网络钩子动态注入到目标Pod 例如， `chaosfs` sidecar 容器运行一个保险丝守护程序来劫持应用程序容器的 I/O 操作。

![Chaos Mesh 工作流](/img/chaos-mesh-workflow.png)

<div class="caption-center"> Chaos Mesh 工作流 </div>

以下是这些组件如何简化混乱测试：

1. 使用YAML文件或 Kubernetes 客户端，用户在 Kubernetes API 服务器上创建或更新chaos 对象。
2. Chaos Mesh 使用 API 服务器监视chaos 对象，并通过创建、更新或删除事件管理chaos 实验的生命周期。 在这个过程中，控制管理人员、chaos-daemon和sidecar容器一起工作以注入错误。
3. 当接纳-webhooks收到一个 Pod 创建请求时，将要创建的 Pod 对象将被动态更新； 例如，它被注入侧边容器和波德。

## 正在运行故障

上述章节介绍了我们如何设计Chaos Mesh以及如何运作。 现在让我们开始做生意，告诉你如何使用Chaos Mesh。 请注意，混乱测试时间可能因待测试申请的复杂性和《CRD》规定的测试时间安排规则而异。

### 环境准备

Chaos Mesh 运行于Kubernetes v1.12 或更高版本。 Helm，Kubernetes包管理工具，部署和管理Chaos Mesh。 在您运行 Chaos Mesh之前，请确保Helm 正确安装在Kubernetes 集群。 为了营造环境，采取以下行动：

1. 请确保您有一个Kubernetes集群。 如果您这样做，请跳转到第2步；否则请使用Chaos Mesh提供的脚本在本地开始：

   ```bash
   // 安装类型
   curl -Lo ./tyn https://github.com/kubernetes-sigs/kind/releases/download/v0. .1/kind-$(uname)-amd64
   chmod +x ./type
   mv . 您的 PATH 类型/some-dir-pATH

   // 获取脚本
   git clone https://github. om/chaos-mesh/chaos-mesh
   cd chaos-mesh
   // 开始数据组
   hack/kind-cluster-build.sh
   ```

   **注意：** 启动 Kubernetes 集群会影响到本地网络故障注入。

2. 如果Kubernetes集群已就绪，请使用 [Helm](https://helm.sh/) 和 [Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) 来部署Chaos Mesh：

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh。 它
   cd chaos-mesh
   // 创建 CRD 资源
   kubectl 应用 -f manif清单/
   // 安装 chaos-mesh
   helm install hem/chaos-mesh --namespace=chaos-teste
   ```

   等待所有组件安装并使用以下方式检查安装状态：

   ```bash
   // 检查chaos-mesh 状态
   kubectl get pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
   ```

   如果安装成功，您可以看到所有的点数都上下和运行。 现在是发挥作用的时候了。

   您可以使用 YAML 定义或 Kubernetes API运行Chaos Mesh

### 使用 YAML 文件运行chaos

您可以通过 YAML 文件方法定义您自己的chaos experience, 这种方法提供了一个快速的 在你部署应用程序后进行混乱实验的方便方式。 若要使用 YAML 文件运行混乱，请按下面的步骤：

**注意：** 为了示例目的，我们使用 TiDB 作为我们正在测试的系统。 您可以使用您选择的目标系统，并相应修改 YAML 文件。

1. 部署名为 `chaos-demo-1` 的TiDB 集群。 您可以使用 [TiDB 运算符](https://github.com/pingcap/tidb-operator) 来部署 TiDB。
2. 创建名为 `kill-tikv.yaml` 的 YAML 文件，并添加以下内容：

   ```yml
   apiVersion: chaos-mesh。 rg/v1alpha1
   种: PodChaos
   元数据:
     名称: pod-kill-chaos-demo
     namespace: chaos-testing
   spec:
     action: pod-kill-kill-chaos-demo
     mode: one
     selector:
       namespaces:
         - chaos-demo-1
       labelector:
         'app. ubernetes.io/component': 'tikv'
     scheduler:
       cron: '@ever 1m'
   ```

3. 保存文件。
4. 若要启动混乱， `kubectl 应用-f kill-tikv.yaml`。

下面的chaos实验模拟在 `chaos-dem-1-` 群组中经常被杀死的 TiKV Podes：

![正在运行Chaos测试](/img/chaos-experiment-running.gif)

<div class="caption-center"> 正在运行Chaos测试 </div>

我们使用系统台程序来监测TiDB集群中的实时QPS变化。 当错误被注入到集群中，QPS会显示一个剧烈的喷射器， 这意味着特定的 TiKV Pod 已被删除，Kubernetes 然后重新创建一个新的 TiKV Pod。

更多的 YAML 文件示例，见 [https://github.com/chaos-mesh/chaos-mesh/tree/master/examps](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。

### 使用 Kubernetes API 运行chaos

Chaos Mesh 使用 CRD 来定义混乱对象，所以您可以直接通过 Kubernetes API来操纵CRD 对象。 这种方式非常方便地将Chaos Mesh应用于您自己的应用，通过自定义的测试场景和自动化混乱试验。

在 [测试-infra](https://github.com/pingcap/tipocket/tree/35206e8483b66f9728b7b14823a10b3e4114e0e3/test-infra) 项目中，我们模拟了 [等](https://github.com/pingcap/tipocket/blob/35206e8483b66f9728b7b14823a10b3e4114e0e3/test-infra/tests/etcd/nemesis_test.go) Kubernetes 上的集群中的潜在错误 包括节点重启、网络故障和文件系统故障。

下面是使用 Kubernetes API的 Chaos Mesh样本脚本：

```
import (
    "context"

 "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
    "sigs.k8s.io/controller-runtime/pkg/client"
)

func main() {
  ...
  延迟 := &chaosv1alpha1.NetworkChaos@un.org
  Spec: chaosv1alpha1。 etworkChaosSpec{...},
      }
      k8sClient := 客户端。 ew(conf, client.Options{ Scheme: scheme.Scheme })
  k8sClient.create(context.TODO(), 延迟)
      k8sClient.Delete(context.TODO(), 延迟)
}
```

## 将来会怎么样？

在这篇文章中，我们向您介绍了我们开放源码云层和本土Chaos Engineering平台Chaos Mesh。 仍然有许多事情在进行中，更详细地说明设计、使用案例和开发情况。 敬请期待。

开放源码只是一个起点。 除了前面各节中提出的基础设施一级的混乱试验。 我们正在支持一系列范围更广的错误类型，例如：

- 在eBPF 和其他工具的帮助下，系统调用和内核级注入错误
- 集成 [故障点](https://github.com/pingcap/failpoint)将特定错误类型注入应用程序函数和语句级别， 它将包括常规注入方法不可能的情景。

今后我们将继续改进Chaos Mesh Dashboard。 这样用户就可以轻松地看到他们在线企业是否以及如何受到故障注入的影响。 此外，我们的路线图包括一个易于使用的故障管道接口。 我们正在计划其他很酷的功能，例如Chaos Mesh Verifier和Chaos Mesh Cloud。

如果您有任何声音感兴趣，与我们一起建立一个世界级Chaos Engineering平台。 让我们的应用程序舞蹈在Kubernetes上发生的混乱！

如果你发现一个错误或认为有些东西丢失，请随时提交一个 [个问题，](https://github.com/chaos-mesh/chaos-mesh/issues), 打开一个 PR, 或在 [CNCF Slack 中的 #project-chaos-mesh频道中给我们发消息](https://slack.cncf.io/) 工作空间。

GitHub: [https://github.com/chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)
