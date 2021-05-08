---
id: get_started_on_minikube
title: 从Minikube开始
---

从 '@site/src/components/PickVersion' 导入选取版本

本文档描述如何使用 Minikube 在 Kubernetes 上部署Chaos Mesh(Linux 或 macOS)

## 必备条件

部署前，请确保 [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) 安装在您的本地机器上。

## 第 1 步：设置 Kubernetes 环境

执行以下步骤来建立当地的Kubernetes环境：

1. 启动一个Kubernetes集群：

   ```bash
   minikube start --kubernetes-version v1.15.0 --cpus 4 --memory "8192mb"
   ```

   > **注：**
   > 
   > 建议使用 `--cpus` 和 `--memory` 标记分配足够的 RAM (超过 8192 MIB) 到虚拟机 (VM)。

2. 安装头盔：

   按照头盔安装步骤: https://helm.sh/docs/intro/install

## 步骤 2: 安装Chaos Mesh

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash
</PickVersion>

上述命令将安装所有CRD，所需的服务账户配置以及所有组件。 在您开始运行混乱实验之前，请验证Chaos Mesh是否正确安装。

您也可以手动使用 [头盔](https://helm.sh/) 到 [安装 Chaos Mesh](../user_guides/installation.md#install-by-helm)。

### 验证您的安装

验证Chaos Mesh 是否在运行

```bash
kubectl get pod -n chaos-testing
```

预期输出：

```bash
命名重命名STATUS RESTARTS AGE
chaos-controller-manager-6d6d95cd94kl8gs 1/1 Running 0 3m40s
chaos-daemon-5shkv 1/1 Running 0 3m40s
chaos-d98856f6-vgrjs 1/1 Running 0 3m40s
```

## 运行Chaos测试

现在你已经在你的环境中部署了Chaos Mesh, 现在是时候用它来进行你的混乱实验了。 跟随 [运行混乱状态实验](../user_guides/run_chaos_experiment.md) 的步骤来运行一个Chaos实验，然后在Chaos Mesh 仪表板上观察它。

## 正在卸载

您可以通过删除命名空间卸载Chaos Mesh。

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --template | kubectl delete -f
</PickVersion>

## 限制

Minikube集群中部署的Chaos Operator已知的一些限制：

- `Netem chaos` 只支持 Minikube 集群 >= 版本 1.6。

在 Minikube 中，默认虚拟机驱动器的图像不包含 `sch_netem` 内核模块。 您可以使用 `没有` 驱动程序(如果您的主机是 `sch_netem` 内核模块已加载) 来尝试使用Minikube 或 [用您自己的 sch_netem 构建一个图像](https://minikube.sigs.k8s.io/docs/contrib/building/iso/)
