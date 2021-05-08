---
id: 安装
title: 安装
---

从 '@site/src/components/PickVersion' 导入选取版本

此文档描述了如何安装Chaos Mesh来对您在 Kubernetes 的应用程序进行混乱试验。

如果您想在膝上型计算机上尝试Chaos Mesh(Linux或macOS)，您可以参考以下两个文档：

- [开始实物版](../get_started/get_started_on_kind.md)
- [从迷你土豆开始](../get_started/get_started_on_minikube.md)

## 必备条件

在部署Chaos Mesh之前，请确保以下物品已经安装：

- Kubernetes 版本 >= 1.12。
- [RBAC](https://kubernetes.io/docs/admin/authorization/rbac) 已启用 (可选)

## Install Chaos Mesh

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash
</PickVersion>

上述命令将安装所有CRD，所需的服务账户配置以及所有组件。 在您开始运行混乱实验之前，请验证Chaos Mesh是否正确安装。

如果您使用 k3s 或 k3d，请同时指定 `--k3 s` 标记。

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash - --k3s
</PickVersion>

> **注：**
> 
> `install.sh` 适合尝试Chaos Mesh 如果你想在生产或其他严重场景中使用Chaos Mesh，Helm 是推荐的部署方法。

### 验证您的安装

验证Chaos Mesh 是否在运行(使用 _kubectl_, 你可以参考 [文档](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands).)

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

## 正在卸载

您可以通过删除命名空间卸载Chaos Mesh。

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --template | kubectl delete -f
</PickVersion>

## 由头盔安装

您也可以通过 [头盔](https://helm.sh) 安装Chaos Mesh。 在你开始安装之前，请确保头盔 v2 或 helm v3 安装正确。

### 第 1 步：将Chaos Mesh仓库添加到Helm repos

```bash
helm repo 添加 chaos-mesh https://charts.chaos-mesh.org
```

添加仓库成功后，您可以通过以下命令搜索可用版本：

```bash
头盔搜索 repo chaos-mesh
```

### 步骤 2: 安装Chaos Mesh

根据您的环境，有两种安装Chaos Mesh的方法：

- 在Docker环境中安装

  1. 创建命名空间 `chaos-测试`:

     ```bash
     kubectl 创建 ns chaos-测试
     ```

  2. 使用头盔安装Chaos Mesh

     - 对于头盔 2.X

     ```bash
     helm install chaos-mesh/chaos-mesh --namespace=chaos-testh
     ```

     - 对于helm 3.X

     ```bash
     helm install chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-treatment
     ```

  3. 检查Chaos Mesh pod是否安装：

     ```bash
     kubectl 获取 pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
     ```

     预期输出：

     ```bash
     命名重命名STATUS RESTARTS AGE
     chaos-controller-manager-6d6d95cd94kl8gs 1/1 Running 0 3m40s
     chaos-daemon-5shkv 1/1 Running 0 3m40s
     chaos-daemon-jpqhd 1/1 Running 0 3m40s
     chaos-daemon-n6mfq 1/1 Running 0 3m40s
     chaos-dashboard-d99856f6-vgrjs 1/1 Running 0 3m40s
     ```

- 在容器环境中安装(kind)

  1. 创建命名空间 `chaos-测试`:

     ```bash
     kubectl 创建 ns chaos-测试
     ```

  2. 使用头盔安装Chaos Mesh

     - 为头盔 2.X

     ```bash
     helm install chaos-mesh/chaos-mesh--namespace=chaos-test-set chaosDaemon.runtime=containerd --set chaemon.socketPath=/run/containerd/containerd.sock
     ```

     - 适合头盔 3.X

     ```bash
     helm install chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-test-set chaosDaemon.runtime=containerd --set chaemon.socketPath=/run/containerd/containerd.sock
     ```

  3. 检查Chaos Mesh pod是否安装：

     ```bash
     kubectl 获取 pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
     ```

     预期输出：

     ```bash
     命名重命名STATUS RESTARTS AGE
     chaos-controller-manager-6d6d95cd94kl8gs 1/1 Running 0 3m40s
     chaos-daemon-5shkv 1/1 Running 0 3m40s
     chaos-daemon-jpqhd 1/1 Running 0 3m40s
     chaos-daemon-n6mfq 1/1 Running 0 3m40s
     chaos-dashboard-d99856f6-vgrjs 1/1 Running 0 3m40s
     ```

- 在容器环境中安装 (k3s)

  1. 创建命名空间 `chaos-测试`:

     ```bash
     kubectl 创建 ns chaos-测试
     ```

  2. 使用头盔安装Chaos Mesh

     - 为头盔 2.X

     ```bash
     helm install chaos-mesh/chaos-mesh--namespace=chaos-test-set chaosDaemon.runtime=containerd --set chaemon.socketPath=/run/k3s/containerd/containerd.sock
     ```

     - 适合头盔 3.X

     ```bash
     helm install chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-test-set chaosDaemon.runtime=containerd --set chaemon.socketPath=/run/k3s/containerd/containerd.sock
     ```

  3. 检查Chaos Mesh pod是否安装：

     ```bash
     kubectl 获取 pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
     ```

     预期输出：

     ```bash
     命名重命名STATUS RESTARTS AGE
     chaos-controller-manager-6d6d95cd94kl8gs 1/1 Running 0 3m40s
     chaos-daemon-5shkv 1/1 Running 0 3m40s
     chaos-daemon-jpqhd 1/1 Running 0 3m40s
     chaos-daemon-n6mfq 1/1 Running 0 3m40s
     chaos-dashboard-d99856f6-vgrjs 1/1 Running 0 3m40s
     ```

在执行上述命令后，您应该能够看到所有Chaos Mesh pods都已上线并运行的输出。 Otherwise, check the current environment according to the prompt message or create an [issue](https://github.com/chaos-mesh/chaos-mesh/issues) for help.
