---
id: 设置_the_development_environment
title: 设置开发环境
sidebar_label: 设置开发环境
---

本文档会让您穿越Chaos Mesh 开发的环境设置过程。

## 必备条件

- [黄金](https://golang.org/dl/) 版本 >= v1.13
- [停靠栏](https://www.docker.com/)
- [gcc](https://gcc.gnu.org/)
- [helm](https://helm.sh/) 版本 >= v2.8.2
- [类型](https://github.com/kubernetes-sigs/kind)
- [节点](https://nodejs.org/en/) and [yarn](https://yarnpkg.com/lang/en/) (用于Chaos Dashboard)

## 准备工具链中

请确保您已满足上述先决条件。 现在遵循下面的步骤来准备编译Chaos Mesh的工具链：

1. 将Chaos Mesh repo 复制到您的本地机器中。

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh.git
   cd chaos-mesh
   ```

2. 安装 Kubernetes API 开发框架 - [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) and [kustomize](https://github.com/kubernetes-sigs/kustomize)。

   ```bash
   确保所有
   ```

3. 请确认 [停靠](https://docs.docker.com/install/) 已经安装并在您的本地机器上运行。

4. 确认 [Docker Regitry](https://docs.docker.com/registry/) 正在运行。 使用注册表地址设置环境变量 `DOCKER_REGISTRY`

   ```bash
   echo 'export DOCKER_REGISTRY=localhost:5000' >> ~/.bash_profile
   source ~/.bash_profile
   ```

5. 确认 `${GOPATH}/bin` 在您的 `PATH` 中。

   ```bash
   echo 'export' pATH=$(go env GOPATH)/bin:${PATH} >> ~/.bash_profile
   ```

   ```bash
   源 ~/.bash_profile
   ```

6. 检查节点相关的环境。

   ```bash
    节点-v
    yarn -v
   ```

现在您可以通过运行来测试工具链：

```bash
制造业：
```

如果输出没有错误，编译工具链将被成功配置。

## 准备部署环境

在工具链准备就绪后，您仍然需要一个本地的Kubernetes集群作为部署环境。 因为这个类型已经安装，您现在可以直接设置 Kubernetes 集群：

```bash
hack/kind-cluster-build.sh
```

上述脚本通过Kind创建了一个Kubernetes集群。 当您使用它时，运行以下命令来删除它：

```bash
删除数据组 --name=type
```

> Bootstrap Chaos 仪表盘。 (可选)
> 
> ```bash
cd ui && yarn
# 运行它
yarn start:default # crossenv REACT_APP_API_URL=http://localhost:2333 BROWSER=no react-脚本开始
```

## 下一步

恭喜！ 您现在都已经为Chaos Mesh 开发而设置。 尝试以下任务：

- [开发新Chaos 类型](dev_hello_world.md)
- [将设施添加到混乱守护进程](add_chaos_daemon.md)
