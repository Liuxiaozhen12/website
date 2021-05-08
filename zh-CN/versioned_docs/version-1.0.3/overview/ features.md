---
id: 功能
title: 功能
sidebar_label: 功能
---

## 简单易用

没有特殊依赖关系，Chaos Mesh 可以轻松地直接部署到 Kubernetes 集群，包括 [Minikube](https://github.com/kubernetes/minikube) and [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 中。

不需要修改测试系统的部署逻辑(SUT) 轻松地将故障注射行为移植到chaos实验中 隐藏基本实现细节，以便用户能够集中精力整理混乱试验。

## Kubernetes设计

Chaos Mesh 使用 [CustomResourceDefinition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CRD) 定义chaos objects.

在Kubernetes领域，CRD是一个成熟的解决方案来执行自定义资源，有大量的执行案例和工具集。 使用 CRD 使Chaos Mesh自然融入Kubernetes 生态系统。
