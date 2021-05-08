---
id: dnschaos_experience
title: DNSChaos 测试
sidebar_label: DNSChaos 测试
---

本文档描述如何在Chaos Mesh中创建 DNSChaos 实验。

DNSChaos 允许您在发送请求后模拟故障DNS响应，如DNS错误或随机IP地址。

## 部署故障的DNS服务

要在 Chaos Mesh中创建 DNSChaos 实验，您需要在Chaos Mesh中部署一个 DNS 服务，执行下面的命令：

```bash
helm upgrade chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-testing --set dnsServer.create=true
```

当部署结束时，检查此DNS服务的状态：

```bash
kubectl get pods -n chaos-testing -l app.kubernetes.io/component=chaos-dns-server
```

请确保Pod的 `STATUS` 是 `正在运行`。

## 配置文件

下面是一个 DNSChaos 配置文件样本：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
型: DNSChaos
metadata :
  name: busybox-dns-chaos
spec:
  随机动作:
  模式:
    - google。 om
    - chaos-网格.
    - github. om
  模式：所有
  选择器：
    命名空间：
      - busybox
  持续时间：'90s'
  scheduler：
    cron：'@每 100'
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。

## 字段描述

- **动作**: 定义DNS chaos的chaos动作。 支持的行动是：

  - `错误` - 发送DNS请求时出错
  - `随机` - 发送DNS请求时获得一个随机IP

- **模式**: 选择要生效的域名, 支持占位符? 和通配符 \*，或指定的域名。

  - 通配符 `_` 必须在字符串的末尾。 例如， `chaos-_.org` 无效。
  - 如果模式为空，将对所有域名生效。

- **选择器**: 指定用于故障注入的目标点. 欲了解更多详情，请参阅 [定义Chaos 实验范围](../user_guides/experiment_scope.md)。

## 注

- 目前，DNSChaos只支持记录类型 `A` and `AAAA`。
- Chaos DNS 服务使用 [k8s_dns_chaos](https://github.com/chaos-mesh/k8s_dns_chaos) 插件运行 CoreDNS 。 如果您的 Kubernetes 集群中的 CoreDNS 服务包含一些特殊配置， 您可以编辑 configMap `dns-server-config` 来使chaos DNS 服务的配置与 K8s CoreDNS 服务的配置一致，如下所示：

  ```bash
  kubectl 编辑 configmap dns-server-config -n chaos-测试
  ```
