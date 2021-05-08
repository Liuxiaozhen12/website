---
id: faqs
title: 常见问题
sidebar_label: 常见问题
---

## 问

### 问：如果我没有部署Kubernetes集群，我能否使用Chaos Mesh来制造混乱？

不，你不能在这种情况下使用Chaos Mesh。 但你仍然可以使用命令行运行混乱试验。 详情参见 [Chaos 命令行使用](https://github.com/pingcap/tipocket/blob/master/doc/command_line_chaos.md)

### 问：我成功地部署了Chaos Mesh并创建了PodChaos实验，但我仍未能创建NetworkChaos/TimeChaos实验。 日志显示如下：

```
2020-06-18T01:05:26.207Z 错误控制器。 imeChaos未能在所有pods {"concilier": "timechaos", "error": "rpc error: code = 不可用desc = 连接错误：desc = \"transport: dialing tcp xx. x.xx.xx：xxxxx：连接：连接拒绝\""}
```

您可以尝试使用参数： `主机网络`，如下所示：

```
# vim helm/chaos-mesh/values.yaml, 更改主机网络从false改为true
hostNetwork: true
```

### 问：我刚刚看到 `个错误：无法获取群集内部kubeconfig：命令 "docker exec --privileged kind-control-fell cat /etc/kubernetes/admin"。 开启失败，错误为：在安装Chaos Mesh时退出状态 1`。 如何修复？

您可以尝试以下命令来修复它：

```
删除数据组
```

然后再次部署。

## Debug

### 问：应用故障后实验无法工作

如下所述，您可以调试：

执行 `kubectl 描述` 来检查指定的chaos 实验资源。

- If there are `NextStart` and `NextRecover` fields under `spec`, then the chaos will be triggered after `NextStart` is executed.

- 如果没有 `下一步开始` 和 `下一步恢复`字段在 `spec`运行下面的命令来获取控制器管理员的日志并查看其中是否有错误。

  ```bash
  kubectl logs -n chaos-testing chaos-controller-manager-xxx (替换为控制器管理器名称) | grep "ERROR"
  ```

  对于错误消息 `没有选择pod`，请运行以下命令来显示标签并检查选择器是否需要。

  ```bash
  kubectl 获取 pods -n 你的命名空间 --show-labels
  ```

如果上述步骤无法解决问题或遇到控制器日志中的其他相关错误， [在 [CNCF Slack](https://slack.cncf.io/) 工作区的 #project-chaos-mesh频道中提交一个问题](https://github.com/chaos-mesh/chaos-mesh/issues) 或消息我们。

## IOChaos

### 问：运行chaosfs sidecar容器失败，日志显示 `pid 文件，确保停泊器不运行或删除 /tmp/fuse/pid`

Chaosfs sidecar容器正在持续重启，您可能会在当前侧边容器中看到以下日志：

```
2020-01-19T06:30:56.629Z INFO chaos-daemon Init hookfs
2020-01-19T06:30:56。 30Z 错误的 chaos-daemon 无法创建pid 文件 {"error": "pid 文件找到 确保停靠码头没有运行或删除 /tmp/fuse/pid"}
github。 om/go-logr/zapr.(*zapLogger).错误
```

- **ause**: Chaos Mesh 使用 Fuse 注入I/O 故障。 如果您指定一个现有目录作为混乱的源路径，它将失败。 当你尝试使用 `保留` 重新收回策略来请求持久性索偿(PVC)资源时，这种情况常常发生。
- **解**: 在这种情况下，使用以下命令将重置策略更改为 `删除` 删除：

```bash
kubectl 补丁pv <your-pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}
```

## 安装

### 问：试图在 OpenShift 中安装chaos-mesh时，跳过了授权问题。

最看起来像这样的消息：

```bash
Error creating: pods "chaos-daemon-" is forbidden: unable
 to validate against any security context constraint: [spec.securityContext.hostNetwork:
 Invalid value: true: Host network is not allowed to be used spec.securityContext.hostPID:
 Invalid value: true: Host PID is not allowed to be used spec.securityContext.hostIPC:
 Invalid value: true: Host IPC is not allowed to be used securityContext.runAsUser:
 Invalid value: "hostPath": hostPath volumes are not allowed to be used spec.containers[0].securityContext.volumes[1]:
 Invalid value: true: Host network is not allowed to be used spec.containers[0].securityContext.containers[0].hostPort:
 Invalid value: 31767: Host ports are not allowed to be used spec.containers[0].securityContext.hostPID:
 Invalid value: true: Host PID is not allowed to be used spec.containers[0].securityContext.hostIPC:
......]
```

您需要添加特色扫描到默认值。

```bash
oc adm policy 添加 scc-to user 特权-n chaos-test-z 默认
```
