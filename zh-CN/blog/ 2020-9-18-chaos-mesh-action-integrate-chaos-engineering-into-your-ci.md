---
slug: /chaos-mesh-action-integrate-chaos-engineering-into your -ci
title: 'chaos-mesh-action: 将 Chaos Engineering 整合到您的 CI'
author: 江省
author_title: Chaos Mesh 贡献者
author_url: https://github.com/WangXiangUSTC
author_image_url: https://avatars3.githubusercontent.com/u/579595?v=4
image: /img/chaos-mesh-action.png
tags:
  - Chaos Mesh
  - Chaos工程
  - GitHub Action
  - CI
---

![chaos-mesh-action - 将Chaos Engineering整合到您的CI](/img/chaos-mesh-action.png)

[Chaos Mesh](https://chaos-mesh.org) 是一个云层本土的混乱测试平台，在Kubernetes环境中精炼chaos。 虽然它在社区中很受欢迎，但它有丰富的故障注入类型和容易使用的仪表盘。 很难使用带有端到端测试或连续集成(CI)的Chaos Mesh。 因此，在系统开发期间出现的问题无法在系统开发之前发现。

在这篇文章中，我将分享我们如何使用chaos-mesh-action，一个GitHub 行动，将Chaos Mesh纳入CI 进程。

<!--truncate-->

chaos-mesh-action 在 [GitHub 市场](https://github.com/marketplace/actions/chaos-mesh)，源代码在 [GitHub](https://github.com/chaos-mesh/chaos-mesh-action) 上。

## 链目动作设计

[GitHub Action](https://docs.github.com/en/actions) 是GitHub 本地支持的 CI/CD 功能， 通过 GitHub 仓库，我们可以轻松地构建自动化和定制的软件开发工作流。

连同GitHub 行动，Chaos Mesh可以更容易地融入系统的日常开发和测试。 从而保证在GitHub 上提交的每个代码都是无漏洞的，并且不会损坏现有代码。 下图显示了融入CI工作流程的chaos-mesh-mesh-action

![CI 工作流中的chaos-mesh-action 集成](/img/chaos-mesh-action-integrate-in-the-ci-workflow.png)

## 在 GitHub 工作流中使用 chaos-mesh-action

[chaos-mesh-action](https://github.com/marketplace/actions/chaos-mesh) 可在 Github 工作流中工作。 一个 GitHub 工作流是一个可配置的自动化过程，您可以在您的仓库中设置它来构建， 测试、包裹、发布或部署任何 GitHub 项目。 要将Chaos Mesh整合到您的 CI 中，请做以下操作：

1. 设计一个工作流。
2. 创建一个工作流。
3. 运行工作流。

### 设计一个工作流

在设计工作流程之前，您必须考虑以下问题：

- 我们将在这个工作流中测试什么功能？
- 我们注入哪种类型的错误？
- 我们如何核实系统的正确性？

作为一个例子，让我们设计一个简单的测试工作流，包括以下步骤：

1. 在 Kubernetes 集群中创建两个Pods。
2. 从另一个Ping one pod
3. 使用 Chaos Mesh 注入网络延时混乱并测试ping命令是否受到影响。

### 创建工作流

在您设计工作流后，下一步是创建它。

1. 导航到包含您想要测试的软件的 GitHub 存储库。
2. 要开始创建一个工作流，请点击 **动作**, 然后点击 **新工作流** 按钮：

![创建工作流](/img/creating-a-workflow.png)

工作流基本上是指按顺序和自动进行的工作的配置。 请注意，任务是在单个文件中配置的。 为了更好地说明问题，我们将脚本分成不同的作业组，如下所示：

- 设置工作流名称和触发规则。

  此作业名称为“Chaos”。 当代码被推送到主分支或一个合并请求被提交到主分支时，将触发此工作流。

  ```yaml
  名称: Chaos

  on:
    pub:
      branches:
        - master
    pull_request:
      branches:
        - master
  ```

- 安装与 CI相关的环境。

  此配置指定了操作系统 (Ubuntu), 并且它使用 [helm/kind-action](https://github.com/marketplace/actions/kind-cluster) 来创建 Kind 集群。 然后，它会输出有关该组的信息。 最后，它将检查 GitHub 仓库以获取工作流。

  ```yaml
  作业:
    build:
      runs-on: ubuntu-latest
      steps:
        - 名称: Creating cluster
          uses: helm/kind-action@v1。 0-rc.

        - 名称：打印集群信息
          运行：|
            kubectl config view
            kubectl cluster info
            kubectl get nodes
            kubectl get pods -n kube-system
            helm version
            kubectl version

        - uses: actions/checkout@v2
  ```

- 部署应用程序。

  在我们的例子中，这个工作应用了一个生成两个Kubernetes Pods的应用程序。

  ```yaml
  - 名称：部署应用程序
       run: |
         kubectl apply -f https://raw.githubusercontent.com/chaos-mesh/apps/master/ping/busybox-statefulset.yaml
  ```

- 带有chaos-mesh-action的混乱状态。

  ```yaml
  - 名称：Run chaos mesh action
      uses: chaos-mesh/chaos-mesh-action@xiang/finforme_script
      env:
        CFG_BASE64: YXBpVmVyc2lvbjogY2hb3MtbWVzaC5vcvcvdjFhbHBoYTEKa2luZDogTmV0d29ya0NoY9zCm1ldGFkYXRhOgogog5hbWU6IG5ldHdvcmstZGVsYXkKICBuYW1lc3BhY2U6IGJ1c3gKc3BlYzoKICBh3Rpb246IGRlb
  ```

  通过chaos-mesh动作，安装Chaos Mesh和注入chaos自动完成。 你只需要准备你想要使用的混乱配置来获取它的 Base64 的代表性。 在这里，我们想要在Pods注入网络延迟混乱，所以我们使用原来的混乱配置如下：

  ```yaml
  apiVersion: chaos-mesh。 rg/v1alpha1
  kind: NetworkChaos
  metadata:
    name: network-delays
    namespace: busybox
  spec:
    action: 延迟# 特定chaos action to inject
    mod: all
    selector:
      pods:
        busybox:
          - busybox-0
    delay:
      latenc: '10ms'
    duration: '5s'
    scheduler:
      cron: '@每 10 s'
    direction: to
    target:
      selector:
        pods:
          busybox:
            - busybox-1
      mod: all
  ```

  您可以使用以下命令获取上述chaos配置文件的 Base64 值：

  ```shell
  $ base64 chaos.yaml
  ```

- 验证系统是否正确。

  在这个工作中，工作流会从另一个Pod向另一个Pod传递，并观察网络延迟的变化。

  ```yaml
  名称：验证
       运行：|
         echo "做一些验证"
         kubectl exec busybox-0-it -n busybox -- ping -c 30 busybox-1. busybox.busybox.svc
  ```

### 运行 Workflow

既然工作流已配置，我们可以通过向主分支提交拉取请求来触发它。 当工作流完成时，验证结果的输出结果看起来与以下内容类似：

```shell
做一些验证
无法使用 TTY - 输入不是终端或正确类型的文件
PING busybox-1。 usybox.busybox.svc (10.244.0.6): 56个数据字节
64 字节从 10.244.0。 : seq=0 tl=63 time=0.069 ms
64 bytes from 10.244.0.6: seq=1 ttl=63 time=10.136 ms
64 bytes from 10 bytes 44.0.6： seq=2 tl=63 time=10.192 ms
64 bytes from 10.244.0.6： seq=3 tl=63 time=10。 29 ms
64 bytes from 10.244.0.6：seq=4 tl=63 time=10.120 ms
64 bytes from 10.244.0。 : seq=5 tl=63 time=0.070 ms
64 bytes from 10.244.0.6: seq=6 tl=63 time=0.073 m
64 bytes from 10 bytes 44.0.6：seq=7 tl=63 time=0.111 ms
64 bytes from 10.244.0.6：seq=8 tl=63 time=0。 70 ms
64 bytes from 10.244.0.6: seq=9 tl=63 time=0.077 ms
……
```

输出表示每次持续约5秒的正常10毫秒延迟。 这与我们注入到chaos-mesh-action中的混乱配置是一致的。

## A. 现状和下一步措施

目前，我们已将chaos-mesh-action 应用到 [TiDB Operator](https://github.com/pingcap/tidb-operator) 项目。 使用 Pod chaos 注入工作流以验证操作员指定实例的重启功能。 其目的是确保在被注射过失随机删除操作者的坑时，tidb操作者能够正常工作。 您可以查看 [TiDB 操作员页面](https://github.com/pingcap/tidb-operator/actions?query=workflow%3Achaos) 了解更多详情。

今后，我们计划在更多的测试中采用chaos-mesh-mesh-action ，以确保TiDB和相关组件的稳定性。 欢迎您使用chaos-mesh-action创建自己的工作流。

如果您发现错误或认为有些东西丢失，请随时提交问题，打开拉取请求(PR)， 或加入我们 [#project-chaos-mesh](https://slack.cncf.io/) 通道在 [CNCF](https://www.cncf.io/) 缺少工作空间。
