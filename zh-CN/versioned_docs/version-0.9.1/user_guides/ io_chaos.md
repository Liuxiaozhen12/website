---
id: iochaos_expert
title: IOChaos Experiment
sidebar_label: IOChaos Experiment
---

本文档帮助您创建 IOChaos 实验。

IOChaos 允许您模拟文件系统错误，如IO 延迟和读/写错误。 当您使用 IO 系统调用时，它可以注入延迟和错误，如 `打开`， `读取` 和 `写入`

> **注：**
> 
> IOChaos只能在应用程序创建之前设置相关标签和注释时使用。 查看 [创建一个混乱状态实验](#create-a-chaos-experiment) 获取更多信息。

## 必备条件

### 应用程序容器的命令和参数

Chaos Mesh 使用 [`等候-fush.sh`](./sidecar_template.md/#tips) 确保在应用程序启动前Fuse-daemon 服务器正常运行。

因此， `等待-fush.sh` 需要注入容器的启动命令。 如果应用程序进程不是由容器 [命令和参数启动](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)启动，IOChaos 无法正常工作。

> **注：**
> 
> 当Kubernetes在未来版本中本地支持 [Sidecar Containers](https://github.com/kubernetes/enhancements/issues/753) 时，我们将删除 `等候.sh` 依赖性。

### 接纳控制器

IOChaos 需要注入一个Sidecar容器到用户pods，而且Sidecar容器可以添加到适用的Kubernetes pods中，使用Chaos Mesh提供的 [突变Webhook 接纳控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)。

> **注：**
> 
> - 虽然默认启用了接纳控制器，但是一些Kubernetes发行版可能会禁用它们。 在这种情况下，按照 [的指示打开接入控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#how-do-i-turn-on-an-admission-controller)。
> - [IOChaos需要校验AdmissionWebhooks](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook) and [MutatingAdmissionWebhooks](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook)

### 模板配置

Chaos Mesh 使用模板机制来简化侧翼注射的配置。

因为 `跳转模板` 与 `helm`发生冲突， 常见模板未包含在 [头盔图表](../installation/installation.md#install-by-helm) 中。 However, it will be deployed automatically if you install Chaos Mesh via the [install script](../installation/installation.md#install-chaos-mesh).

默认情况下，通用模板配置地图应该部署在与Chaos Mesh相同的命名空间中。

```bash
kubectl 应用 -f 清单/chaosfs-sidecar.yaml -n <Chaos Mesh namespace>
```

### 数据目录

目标pod 中应用程序的数据目录应该是 `PersistentVolumes</strong> 的 <strong x-id="1">子目录`。

示例：

```yaml
# 关于 tikv PersistentVolumes
volumounts:
  - 名称: datadir
    mountPath: /var/lib/tikv

# 要启动tikv
ARGS="--pd=${CLUSTER_NAME}-pd:2379
  --adverse-addr=${HOSTNAME}${HEADLESS_SERVICE_NAME}。${NAMESPACE}.svc:20160 \
  --addr=0.0.0。 :20160 \
  --data-dir=/var/lib/tikv/data \ # 数据目录
  --capacity=${CAPACITY} \
  --config=/etc/tikv/tikv.toml
```

> **注：**
> 
> - TiKV的默认数据目录不是 `PersistentVolumes` 的子目录。
> - 如果您正在测试一个 TiDB 集群，您需要在 [`_start_tikv.sh.tpl`](https://github.com/pingcap/tidb-operator/blob/master/charts/tidb-cluster/templates/scripts/_start_tikv.sh.tpl) 修改它。
> - PD 与 TiKV 有同样的问题。 您需要在 [`_start_pd.sh.tpl`](https://github.com/pingcap/tidb-operator/blob/master/charts/tidb-cluster/templates/scripts/_start_pd.sh.tpl) 修改PD的数据目录。

## 注入配置

注入配置是另一个 [配置映射](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) ，需要它来完成IO Chaos。

要在开始您的chaos实验前定义一个指定的应用程序配置映射，请参阅此 [文档](sidecar_configmap.md)。

您可以通过以下命令将为您的应用程序定义的 ConfigMap应用到 Kubernetes 集群：

```bash
kubectl 应用程序 -f configmap.yaml # app-configmap.yaml 是配置地图文件
```

## 配置文件

下面是IOChaos的 YAML 文件样本：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: IoChaos
metadata:
  name: io-delayy-example
  namespace: chaos-testing
spec:
  action: mixed
  mod: one
  duration: '400s'
  path: ''
  selector:
    label Selector:
      'app. 伯尔尼。 o/component: 'tikv'
  layer: 'fs'
  percent: '50'
  delay: '1ms'
  scheduler:
    cron: '@ever 10m'
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。
定义 IO 动作返回的错误代码。 此值和 Linux 系统</a> 定义的 错误是一致的。 当您选择 `错误` 或 `混合` 动作时，需要设置此字段。 如果 `错误` 为空，操作员随机生成了一个错误代码。 查看 [常见的Linux 系统错误](#common-linux-system-errors) 获取更多Linux 系统错误代码。</td> 

</tr> 

</tbody> </table> 



## 用法

在应用程序创建之前，您需要在应用程序命名空间启用接纳-webhook：



```bash
kubectl 创建ns app-ns
kubectl 标签ns app-ns admission-webhook=enabled
```


然后，我们有两种方法来标记我们想要注入IO Chaos的点：

1. 在命名空间中设置注解 `admission-webhook.chaos-mesh.org/init-request` ，然后这个命名空间中的所有点都将被注入选择器要求。



```bash
# 设置注解
kubectl 注解 ns app-ns admission-webhook.chaos-mesh.org/init-request=chaosfs-tikv

# 创建您的应用程序
...
```


2. 在pods设置注解 `admission-webhook.chaos-mesh.org/request` ，您可以选中 [示例](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/etcd/etcd.yaml)

然后，您可以启动应用程序并定义 YAML 文件来开始您的混乱。



> **注意**:
> 
> 上述示例中注解的值 `chaos-tikv` 是您注入配置中的名称字段。



### 开始一个混乱测试

假设您正在使用 `示例/io-mixedexample.yaml`，您可以运行以下命令来创建一个chaos实验：



```bash
kubectl 应用 -f 示例/io-mixedexample.yaml
```




## IOChaos 可用的操作

IOChaos目前支持以下行动：

- **延迟**: IO 延迟操作 您可以在IO 操作返回结果之前指定延迟。
- **错误**: IO 错误操作 在此模式下，读写/写入IO 操作返回一个错误。
- **混合**: **延迟** 和 **错误** 动作。



### 延迟

如果您正在使用延迟模式，您可以编辑速度如下：



```yaml
示例：
  操作：延迟
  延迟：“1毫秒”
```


If `delay` is not specified, it is generated randomly on runtime.



### 错误

如果您正在使用错误模式，您可以编辑速度如下：



```yaml
示例：
  操作：错误
  错误: '32'
```


If `errno` is not specified, it is generated randomly on runtime.



### 混合的

如果您正在使用混合模式，您可以编辑速度如下：



```yaml
示例：
  操作：混合
  延迟：“1毫秒”
  错误：“32”
```


混合模式定义了 **延迟** 和 **错误，在一个选票中定义了** 动作。



## 常见的 Linux 系统错误

常见的Linux系统错误如下：

- `1`: 操作不被允许
- `2`: 没有这样的文件或目录
- `5`: I/O 错误
- `6`: 没有这样的设备或地址
- `12`: 内存不足
- `16`: 设备或资源繁忙的
- `17`: 文件存在
- `20`: 不是一个目录
- `22`: 无效参数
- `24`: 太多打开的文件
- `28`: 设备上没有剩余空间

参考 [个错误：Linux 系统错误](https://www-numi.fnal.gov/offline_software/srt_public_context/WebDocs/Errors/unix_system_errors.html) 个更多。



## 可用的方法

现有方法如下：

- `打开`
- `已读`
- `写`
- `mkdir`
- `rmdir`
- `opendir`
- `fsync`
- `刷入`
- `发布`
- `截图`
- `getattr`
- `聊天室`
- `chmod`
- `utimens`
- `分配`
- `getlk`
- `setlk`
- `setlkw`
- `状态`
- `读取链接`
- `symlink`
- `创建`
- `访问`
- `链接`
- `mknod`
- `重命名：`
- `取消链接`
- `getxattr`
- `listxattr`
- `removexattr`
- `setxattr`
