---
id: Audios_experience
title: AwsChaos Experiment
sidebar_label: AwsChaos Experiment
---

本文档介绍了如何创建 AwsChaos 实验。

AwsChaos可以帮助您将故障注入到指定的 AWS 实例中，特别是 `ec2-stop`, `ec2-重启` 和 `分离音量`。

- **Ec2 停止** 动作定期停止指定的 ec2 实例。

- **Ec2 重启** 动作定期重启指定的 ec2 实例。

- **将音量** 动作从指定的 ec2 实例中分离存储音量。

## 秘密文件

为了便利与特设工作组集群的连接， 您可以首先创建一个 kubernetes 秘密文件来存储相关信息(例如访问密钥id)。

下面是示例 `秘密` 文件：

```yaml
apiVersion: v1
kind: Secret
metdata :
  name: cloud-key-secret
  namespace: chaos-testing
type: Opaque
stringData:
  aws_access_key_id: your-aws-access-key-id
  aws_secret_access_key: your-aws-secret-access-key
```

- **name** 定义了 kubernetes 秘密的名称。
- **命名空间** 定义了kubernetes 秘密的命名空间。
- **aws_access_key_id** 存储您的 AWS 访问密钥ID。
- **aws_secret_access_key** 存储您的 AWS 秘密访问密钥。

## `ec2-stop` 配置文件

下面是示例 `ec2-stop` 配置文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: AwsChaos
metadata:
  name: ec2-stop-example
  namespace: chaos-test
spec:
  action: ec2-stop
  secret名称: 'cloud-key-secret'
  awsRegion: 'us-east-2'
  ec2Instance: 'your-ec2-instance-id'
  duration: '5m'
  scheduler:
    cron: '@ever 10m'
```

关于停止ec2实例的更多详情，请参阅 [docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Stop_Start.html)。

配置模板中每个字段的详细描述，请参阅 [`字段描述`](#fields-description)。

## `ec2-重启` 配置文件

下面是示例 `ec2 重启` 配置文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: AwsChaos
metadata:
  name: ec2-resturn example
  namespace: chaos-testing
spec:
  action: ec2-rehont
  secret名称: 'cloud-key-secret'
  awsRegion: 'us-east-2'
  ec2Instance: 'your-ec2-instance-id'
  duration: '5m'
  scheduler:
    cron: '@ever 10m'
```

关于重启ec2实例的更多详情，请参阅 [docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-reboot.html)。

配置模板中每个字段的详细描述，请参阅 [`字段描述`](#fields-description)。

## `分离音量` 配置文件

下面是一个示例 `分离音量` 配置文件：

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: AwsChaos
metadata:
  name: ec2-detach-volume-example
  namespace: chaos-testing
spec:
  action: ec2-stop
  secretName: 'cloud-key-secret'
  awsRegion: 'us-east-2'
  ec2Instance: 'your-ec2-instance-id'
  volumeID: 'your-volume-id'
  deviceName: '/dev/sdf'
  duration: '5m'
  scheduler:
    cron: '@every 10m'
```

欲了解更多关于从Amazon EBS中分离出一卷的详情，请参阅 [docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-detaching-volume.html)。

配置模板中每个字段的详细描述，请参阅 [`字段描述`](#fields-description)。

## 字段描述

- **动作** 定义了特定的 AWS 实例的 chaos 操作。 支持的动作： `ec2-stop` / `ec2-重启` / `分离音量`。
- **secretName** 定义了用于存储 AWS 信息的 kubernetes 秘密名称。
- **awsregion** 定义了 AWS 区域。
- **ec2实例** 表示了 ec2 实例的 ID。
- **音量ID** 需要在 `独立音量` 动作。 它表示EBS卷的 ID。
- **设备名称** 需要在 `独立音量` 动作。 它表示设备的名称。
- **持续时间** 定义了每次混乱状态实验的持续时间。
- **调度器** 定义了混乱状态实验运行时间的调度规则。 欲了解更多规则信息，请参阅 [robfig/cron](https://godoc.org/github.com/robfig/cron)。
