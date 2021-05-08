---
id: 运行chaos_expert
title: 运行Chaos Experiment
---

从 '@site/src/components/PickVersion' 导入选取版本

现在你已经在你的环境中部署了Chaos Mesh, 现在是时候用它来进行你的混乱实验了。 这个文档使您走过了运行混乱的实验过程。 报告还介绍了经常性的混乱试验行动。

## 第 1 步：部署目标组

第一步总是部署一个测试组。 为示例目的， [webshow](https://github.com/chaos-mesh/web-show) 被用作示例集群，因为它允许我们直接观察网络混乱的影响。 您也可以部署自己的测试应用程序。

<PickVersion className="language-bash">
  curl -SSL https://mirrors.chaos-mesh.org/latest/web-show/dep.sh | sh
</PickVersion>

在执行上面的命令后，您可以在浏览器中访问 [`http://localhost:8081`](http://localhost:8081) 查看网页显示应用程序。

> **注：**
> 
> 如果在服务器上部署了网络节目，您需要使用主机IP访问应用程序。

## 步骤 2: 定义实验配置文件

在YAML文件中定义了chaos实验配置。 您需要根据下面样本中可用的字段创建您自己的实验配置文件：

<!-- prettier-ignore -->
```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
种: NetworkChaos
metadata:
  name: web-show-network延迟
spec:
  action: 延迟# 特定的chaos 操作注入
  模式: 一个 # 运行chaos 操作的模式; 支持的模式是一次/全部/固定/固定/随机-最大百分比
  选择器：# 播出混乱动作的地点的点数
    命名空间：
      - 默认
    标签选择器：
      "应用": "web-show" # 用于混乱的点的标签
  延迟：
    延迟: "10ms"
  持续时间: "30" # 注入chaos 实验的持续时间
  时间安排: # 关于马铃薯试验的运行时间的调度规则.
    cron：“@每 60 ”
```

## 第 3 步：应用一个chaos 测试

运行以下命令来应用实验：

```bash
# 请确保您在chaos-mesh/examples/web-show 目录
kubectl apply -f network-delay.yaml
```

然后您可以在浏览器中访问 [`http://localhost:8081`](http://localhost:8081) 来检查混乱实验的结果。

![网络延迟](/img/using-chaos-mesh-to-insert-delays-in-web-show.png)

从线条图表，您可以每隔60秒就知道有10毫秒网络延迟。 如果您被触发并想尝试更多Chaos Mesh的混乱实验，请查看 [示例/web-show](https://github.com/pingcap/chaos-mesh/tree/master/examples/web-show)。

## 3. 混乱试验方面的定期行动

在本节中，当混乱状态试验正在运行时，您将学习一些后续操作。

### 更新一个Chaos expert

```bash
vim network-delay.yaml # 修改 network-delay.yaml 到您想要的
kubectl 应用 -f network-delay.yaml
```

### 暂停一个混乱测试

```bash
kubectl 注释 networkchaos web-show-modification experiment.chaos-mesh.org/pause=true
```

### 恢复一个混乱测试

```bash
kubectl 注释 networkchaos web-show-modification experiment.chaos-mesh.org/pause-
```

### 删除一个混乱测试

```bash
kubectl 删除 -f network-delay.yaml
```

如果您遇到删除操作被阻止的情况，这意味着一些目标点无法恢复。 您可以检查Chaos Mesh的日志，或只是感觉可以随时提交一个 [问题](https://github.com/pingcap/chaos-mesh/issues)。 此外，您还可以通过以下命令强制删除混乱测试：

```bash
kubectl 批注网络chaos web-show-network-delays chaos-mesh.chaos-mesh.org/cleanFinalizer=forced
```

### 在 Chaos 仪表盘中观看您的chaos测试

Chaos 仪表板是一个用于管理、设计、监测Chaos 实验的Web UI。 敬请给予更多的支持，或与我们一道促成这种支持。

> **注：**
> 
> 如果未安装Chaos 仪表板，通过执行 `helm 升级chaos-mesh chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-testing --set dashboard.create=true` 来升级Chaos Mesh。

访问它的典型方式是使用 `kubectl 端口-forward`:

```bash
kubectl port-forward - n chaos-testing svc/chaos-ashboard 2333:2333
```

然后您可以在浏览器中访问 [`http://localhost:2333`](http://localhost:2333)

若要快速查看Chaos 仪表板工作流程，请查看以下文章：

- [Craig Morten： K8s Chaos Dive： Chaos-Mesh Part 1](https://dev.to/craigmorten/k8s-chaos-dive-2-chaos-mesh-part-1-2i96)
