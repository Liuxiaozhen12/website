---
slug: /run_your_first_chaos_expert
title: 在 10 分钟内运行您的第一次迷你测试
author: Cwen Yin
author_title: Chaos Mesh 维护者
author_url: https://github.com/cwen0
author_image_url: https://avatars1.githubusercontent.com/u/22956341?v=4
image: /img/run-first chaos-experimentation in ten-minutes.jpg
tags:
  - Chaos Mesh
  - Chaos工程
  - Kubernetes
---

![在10分钟内运行您的第一次混乱测试](/img/run-first-chaos-experiment-in-ten-minutes.jpg)

Chaos Engineering是一种通过模拟异常或破坏性条件来测试生产软件系统强度的方法。 然而，对许多人来说，从学习Chaos Engineering转为在自己的系统上实践Chaos Engineering是令人生畏的。 这听起来就象需要一个装备齐全的小组来规划前面的大想法之一。 好吧，它不必如此。 要开始制造混乱，您可能只是一个合适的平台。

<!--truncate-->

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh) 是一个 **易于使用的**, 打开源头、云层土著Chaos Engineering play, 可以在Kubernetes 环境中调节chaos。 这个十分钟的教程将帮助您快速开始Chaos Engineering并运行您第一次Chaos Mesh的混乱试验。

关于Chaos Mesh的更多信息，请参阅我们先前的 [篇文章](https://pingcap.com/blog/chaos-mesh-your-chaos-engineering-solution-for-system-resiliency-on-kubernetes/) 或 [chaos-mesh项目](https://github.com/chaos-mesh/chaos-mesh) 在 GitHub 上。

## 我们小实验的预览

混乱实验类似于我们在科学课堂上进行的实验。 在受控制的环境中刺激动荡的局面是非常好的。 在我们这里的情况下，我们将在一个名为 [web-show](https://github.com/chaos-mesh/web-show) 的小型网页应用程序上模拟网络混乱。 为了使混乱效应直观， 网页显示每隔10秒记录从其pod 到kube-controller pod 的延迟(在 `kube-system`的命名空间下)。

以下片段显示了安装Chaos Mesh、部署websh、在几个命令中创建chaos实验的过程：

![整个混乱试验进程](/img/whole-process-of-chaos-experiment.gif)

<div class="caption-center"> 整个混乱试验进程 </div>

现在是你回来了！ 现在是获得你的手的脏话的时候了。

## 让我们开始吧！

对于我们的简单实验，我们在 Docker ([Kind](https://kind.sigs.k8s.io/))中使用 Kubernetes 进行开发。 您可以随时使用 [Minikube](https://minikube.sigs.k8s.io/) 或任何现有的 Kubernetes 集群来跟随。

### 准备环境

在前进之前，请确保您有 [Git](https://git-scm.com/) and [Docker](https://www.docker.com/) 安装在您的本地计算机上，并使用 Docker 启动和运行。 对于macOS，建议至少分配6个CPU核心到 Docker。 欲了解详情，请见 [Mac 停靠配置](https://docs.docker.com/docker-for-mac/#advanced)。

1. 获取Chaos Mesh：

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh.git
   cd chaos-mesh/
   ```

2. 使用 `安装.sh` 脚本安装Chaos Mesh

   ```bash
   ./install.sh --本地类型
   ```

   `install.sh` 是一个自动化的 shell 脚本，用于检查您的环境，安装 Kind，启动本地的 Kubernetes 集群，并部署Chaos Mesh。 要查看 `install.sh`的详细描述，您可以包含 `--help` 选项。

   > **注：**
   > 
   > 如果您的本地计算机无法从 `docker.io` 或 `gcr拉取图像。 o`, 使用本地 gcr.io 镜像并执行 `./install.sh --local type --docker-mirror` 代替。

3. 设置系统环境变量：

   ```bash
   源 ~/.bash_profile
   ```

> **注：**
> 
> - 根据您的网络，这些步骤可能需要几分钟。
> - 如果您看到像这样的错误信息：
>     
>     `bash
  错误: 创建集群失败: 生成 kubeadm config 内容失败: 从节点获取Kubernetes 版本失败: 获取文件失败: 命令 "docker exec --private kind-control cat /kind/version" 失败: 退出状态 1`
>     
>     在本地计算机上增加停靠站点的可用资源并执行以下命令：
>     
>     `bash
  ./install.sh --local type --force-local-kube`

当进程完成后，您将看到一个指示Chaos Mesh的消息已成功安装。

### 部署应用程序

下一步是部署测试应用程序。 就我们在这里的情况而言，我们选择了网络节目，因为它使我们能够直接观察网络混乱的影响。 您也可以部署自己的测试应用程序。

1. 使用 `部署.sh` 脚本部署网络显示：

   ```bash
   # 请确保您在Chaos Mesh 目录
   cd 示例/webshow &&
   ./dep.sh
   ```

   > **注：**
   > 
   > 如果您的本地计算机无法从 `docker.io`中拉取图像，请使用 `本地gcr。 o` 镜像并执行 `./dep.sh --docker-mirror` 代替。

2. 访问网页显示应用程序。 从您的浏览器转到 `http://localhost:8081`。

### 创建混乱测试

现在一切都已准备好，现在是运行你的混乱状态实验的时候了！

Chaos Mesh 使用 [CustomResourceDefinition](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) (CRD) 来定义混乱试验。 CRD对象是根据不同的实验假设单独设计的，这大大简化了CRD对象的定义。 目前，在Chaos Mesh实施的CRD对象包括PodChaos、NetworkChaos、IOChaos、TimeChaos和KernelChaos。 稍后，我们会支持更多的故障注入类型。

在此实验中，我们正在使用 [NetworkChaos](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/web-show/network-delay.yaml) 进行chaos 试验。 用 YAML 编写的 NetworkChaos 配置文件显示如下：

```
apiVersion: chaos-mesh。 rg/v1alpha1
kind: NetworkChaos
metdata:
  name: network-delayy-example
spec:
  action: delays
  mode: one
  selector:
    namespace:
      - default
    labelecators:
      "app": "web-show"
  delay:
    latenc: "10ms"
    correlation: "100"
    jitter: "0ms"
  dur: "30s"
  scheduler:
    cron: "@every 60"
```

关于 NetworkChaos 动作的详细描述，见 [Chaos Mesh wiki](https://github.com/chaos-mesh/chaos-mesh/wiki/Network-Chaos)。 在这里，我们刚刚将配置修改为：

- target: `web-show`
- 任务：注入 `10ms` 网络延时每 `60s`
- 攻击持续时间：每次 `30秒`

要启动NetworkChaos，请执行以下操作：

1. 运行 `network-delay.yaml`:

   ```bash
   # 请确保您在chaos-mesh/examples/web-show 目录
   kubectl apply -f network-delay.yaml
   ```

2. 访问网页显示应用程序。 在 web 浏览器中，转到 `http://localhost:8081`。

   从线条图表，您可以每隔60秒就知道有10毫秒网络延迟。

![使用 Chaos Mesh 在网络节目中插入延迟](/img/using-chaos-mesh-to-insert-delays-in-web-show.png)

<div class="caption-center"> 使用 Chaos Mesh 在网络节目中插入延迟 </div>

恭喜！ 你只是挑起了一点混乱。 如果您被触发并想尝试更多Chaos Mesh的混乱实验，请查看 [示例/web-show](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples/web-show)。

### 删除Chaos expert

一旦你完成测试，结束混乱试验。

1. 删除 `network-delay.yaml`:

   ```bash
   # 请确保您在chaos-mesh/examples/web-show 目录
   kubectl delete -f network-delay.yaml
   ```

2. 访问网页显示应用程序。 从您的浏览器转到 `http://localhost:8081`。

从线路图表，您可以看到网络延迟级别恢复到正常水平。

![网络延迟级别恢复到正常状态](/img/network-latency-level-is-back-to-normal.png)

<div class="caption-center"> 网络延迟级别恢复到正常状态 </div>

### 删除 Kubernetes 数据组

在你完成了chaos实验后，执行以下命令来删除Kubernetes集群：

```bash
删除数据组 --name=type
```

> **注：**
> 
> 如果遇到 `种：命令找不到` 错误，请先执行 `源 ~/.bash_profile` 命令，然后删除 Kubernetes 集群。

## 酷！ 接下来是什么？

祝贺您第一次成功地进入Chaos Engineering。 它是如何感觉的？ Chaos Engineering很容易，对吗？ 但也许Chaos Mesh并不容易使用。 命令行操作不方便，手动写入 YAML 文件有点烦琐，或者检查实验结果有点模糊？ 别担心，Chaos 仪表板正在运行！ 在 web 上运行混乱的实验可以让人兴奋！ 如果你想帮助我们建立云端平台测试标准，或者让Chaos Mesh 更好，我们很想听到你的消息！

如果您发现错误或认为有些东西丢失，请随时提交问题，打开拉取请求(PR)， 或加入我们在 [CNCF 缺少工作区的 #project-chaos-mesh频道](https://slack.cncf.io/)。

GitHub: [https://github.com/chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)
