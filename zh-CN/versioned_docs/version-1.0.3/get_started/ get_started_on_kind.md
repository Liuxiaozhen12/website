---
id: get_started_on_type
title: 开始实物版
---

从 '@site/src/components/PickVersion' 导入选取版本

本文档描述如何在您的膝上型电脑(Linux或macOS)上部署Chaos Mesh。

## 必备条件

在部署前，请确保 [停靠](https://docs.docker.com/install/) 已经安装并在您的本地机器上运行。

## Install Chaos Mesh

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash - --local 类型
</PickVersion>

`安装。 h` 是一个帮助您安装依赖关系的自动化shell脚本，如 `kubectl`, `头盔`, `类型`, 和 `kubernetes`, 并部署Chaos Mesh本身。

在执行上述命令后，您需要验证Chaos Mesh是否正确安装。

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

<PickVersion className="language-bash">
  curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --template | kubectl delete -f
</PickVersion>

此外，您还可以直接删除命名空间来卸载Chaos Mesh。

```bash
kubectl 删除 ns chaos-测试
```

## 清理数据组

```bash
删除数据组 --name=type
```
