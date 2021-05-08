---
id: sidecar_configmap
title: Sidecar ConfigMap
sidebar_label: Sidecar ConfigMap
---

此文档引导您为您的应用程序定义一个指定的边形配置图。

## 为什么我们需要指定的Sidecar配置地图？

Chaos Mesh 在 [sidecar 容器](https://www.magalix.com/blog/the-sidecar-pattern) 运行一个 [fuse-daemon](https://www.kernel.org/doc/Documentation/filesystems/fuse.txt) 服务器来实现文件系统 IOChaos。

在sidecar容器中，fuse-daemon需要在应用程序启动前由 [fusermount](http://manpages.ubuntu.com/manpages/bionic/en/man1/fusermount.1.html) 挂载应用程序的数据目录。

## 它是如何运作的？

目前，Chaos Mesh 支持两种配置地图：

1. 模板配置。 每个侧边栏配置的骨架相似，以便满足不同的要求并简化配置， Chaos Mesh 支持创建用于不同应用程序的常见模板。 关于模板配置的详情，请参阅 [模板配置](sidecar_template.md)。

2. 注入配置 此配置将与模板配置合并，最后生成一个配置以注入选定的pods。 由于大多数应用程序使用不同的数据目录，容量名称或容器名称 您可以根据在第一步创建的共同模板定义不同的参数。

## 注入配置

以下内容是为tikv定义的注入配置图：

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  nameosfs-tikv
  namespace: chaos-teste
  标签:
    appp. 伯尔尼。 o/component: webhook
data :
  chaosfs-tikv: |
    name: chaosfs-tikv
    selector:
      label Selector:
        "app. 伯尔尼。 o/component: "tikv"
    template: chaosfs-sidecar
    参数:
      ContainerName: "tikv"
      DataPath: "/var/lib/tikv/data"
      MountPath: "/var/lib/tikv"
      VolumeName: "tikv"
```

注入配置定义了一些不同应用程序的注入参数，它基于之前创建的通用模板。

对于此配置中定义的字段，我们有以下一些简短的描述：

- **名称**: 注入配置名称，独特地在一个命名空间中识别注入配置。 然而，您可以在不同的命名空间中使用相同的名称，因此这对于实现多租户是有用的。
- **选择器**: 用于过滤点到侧边。
- **模板**: 用于渲染注入配置的模板配置名称。 “chaosfs-sidecar”模板用于注射fus-serversidecar。
- **参数**: 您应该定义在模板中使用的参数。

欲了解更多示例配置地图文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/release-0.9/examples/chaosfs-configmap)。

## 用法

查看 [IOChaos 文档](io_chaos.md)。
