---
id: 实验范围
title: 定义Chaos实验的范围
sidebar_label: 定义Chaos实验的范围
---

本文件介绍了如何界定混乱试验的范围。

Chaos Mesh提供了各种选择器，您可以用来定义您的混乱实验范围。 这些选择器是在 `spec.selector` chaos object中定义的。

## 命名空间选择器

命名空间选择器通过命名空间筛选chaos实验目标。 定义为一组字符串。 Chaos Mesh的默认命名空间选择器是chaos实验对象。 例如：

```yaml
示例：
  选择器：
    命名空间：
      - “app-ns”
```

## 标签选择器

标签选择器筛选器通过标签筛选实验目标。 定义为字符串键和值的映射。 例如：

```yaml
示例：
  选择器：
    标签选择器：
      'app.kubernetes.io/component': 'tikv'
```

## 批注选择器

批注选择器过滤批量测试目标。 定义为字符串键和值的映射。 例如：

```yaml
示例：
  选择器：
    批注选择器：
      '示例注释'：'group-a'
```

## 字段选择器

字段选择器过滤按资源字段的实验目标。 定义为字符串键和值的映射。 例如：

```yaml
示例：
  选择器：
    field Selector：
      'metadata.name'：'my-pod'
```

欲了解更多字段选择器的详情，请参阅 [Kubernetes文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/)。

## Pod 相位选择器

Pod Phase 选择器筛选器按条件筛选测试目标。 定义为一组字符串。 支持的条件： `待处理`, `运行`, `成功`, `失败`, `未知`. 例如：

```yaml
示例：
  选择器：
    podPhaseSelector：
      - '运行'
```

## Pod 选择器

Pod 选择器过滤剧情测试目标。 定义为字符串键和值的映射。 此地图中的密钥指定了pod所属的命名空间，而密钥下的每个值都是pod。 如果此选择器不是空的，则此地图中定义的这些诗句将直接使用，其它定义的选择器将被忽略。 例如：

```yaml
spec:
  选择器:
    pods:
      tidb-cluster: # 目标点的命名空间
        - basic-tidb-0
        - basic-pd-0
        - basic-tikv-0
        - basic-tikv-1
```
