---
id: develop_a_new_chaos
title: 开发一个新的Chaos
sidebar_label: 开发一个新的Chaos
---

[准备开发环境](setup_env.md)后，让我们开发一种新型的混乱，HelloWorldChaos，它只能打印一个“你好的世界！ 消息到日志。 一般来说，要为Chaos Mesh添加一个新的混乱类型，您需要采取以下步骤：

1. [定义schema类型](#define-the-schema-type)
2. [注册CRD](#register-the-crd)
3. [注册此混乱对象的处理程序](#register-the-handler-for-this-chaos-object)
4. [制作Docker图像](#make-the-docker-image)
5. [运行故障](#run-chaos)

## 定义schema类型

要定义新的chaos对象的方案类型，请添加 `helloworldchaos_types。 o` 在 api 目录 [`api/v1alpha1`](https://github.com/chaos-mesh/chaos-mesh/tree/master/api/v1alpha1) 中填写以下内容：

```go
package v1alpha1

import (
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// +kubebuilder:object:root=true
// +chaos-mesh:base

// HelloWorldChaos is the Schema for the helloworldchaos API
type HelloWorldChaos struct {
    metav1.TypeMeta   `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`

    Spec   HelloWorldChaosSpec   `json:"spec"`
    Status HelloWorldChaosStatus `json:"status,omitempty"`
}

// HelloWorldChaosSpec is the content of the specification for a HelloWorldChaos
type HelloWorldChaosSpec struct {
    // Duration represents the duration of the chaos action
    // +optional
    Duration *string `json:"duration,omitempty"`

    // Scheduler defines some schedule rules to control the running time of the chaos experiment about time.
    // +可选
    调度器 *SchedulerSpec `json:"scheduler, mitempty"
}

// HelloWorldChaosStatus 代表了HelloWorldChaos
类型HelloWorldChaosStatus struct Windows
    ChaosStatus `json:”， nline"
}
```

添加此文件后，已定义HelloWorldChaos schema类型。 它的结构可以描述为下面的 YAML 文件：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
种: HelloWorldChaos
metadata :
  name: <name-of-this-resource>
  namespace: <ns-of-this-resource>
spec:
  duration: <duration-of-every-action>
  scheduler:
    cron: <the-cron-job-definition-of-this-chaos>
status:
  stage : <phase-of-this-resource>
...
```

`生成` 将生成它所需要的锅炉功能，用于集成Chaos Mesh中的资源。

## 注册CRD

HelloWorldChaos对象是Kubernetes的自定义资源对象。 这意味着您需要在 Kubernetes API中注册相应的 CRD。 运行 `make yaml`, 然后在 `config/crd/bases/chaos-mesh.org_helloworldchaos.yaml` 中生成CRD。 为了将所有这些YAML文件合并到 `manifests/crd.yaml`, 通过添加相应的行来修改 [kustomization.yaml](https://github.com/chaos-mesh/chaos-mesh/blob/master/config/crd/kustomization.yaml) 如下所示：

```yaml
资源：
  - bases/chaos-mesh.org_podchaos.yaml
  - bases/chaos-mesh.org_networkchaos.yaml
  - bases/chaos-mesh.org_iochaos.yaml
  - bases/chaos-mesh.org_helloworldchaos.yaml # 这是新行
```

再次运行 `make yaml` ，HelloWorldChaos 的定义将在 `manifests/crd.yaml` 中显示。 您可以通过 `git diff` 检查它

## 注册此混乱对象的处理程序

创建文件 `控制器/helloworldchaos/types.go` 并填写以下代码：

```go
软件包 helloworldchaos

导入 (
    "context"

    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s. o/controller-runtime”

    "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
    "github.com/chaos-mesh/chaos-mesh/pkg/router"
    ctx "github. om/chaos-mesh/chaos-mesh/pkg/router/context”
    结尾“github.com/chaos-mesh/chaos-mesh/pkg/router/endpoint”
)

类型端点结构 {
    ctx.Context
}

func (e *endpoint) Appy(ctx context ontext, req ctrl.Request, chaos v1alphalpha1.InnerObject) 错误。
    e.Log.Info("Hello World!")
    返回 nil
}

func (ae *endpoint) Recovery (ctx context, req ctrl.Request, chaos v1alpha1. nnerObject) 错误 {
    return nil
}

func (e *endpoint) Object() v1alphalpha1.InnerObject Group
    return &v1alpha1. elloWorldChaos{}
}

func init() v.
    router.Register("helloworldchaos", &v1alpha1.HelloWorldChaos{}, function(obj runtime). bject) bool {
        return true
    }, function(ctx ctx.Context) 结束 ndpoint v.
        return &endpoint{
            Context: ctx,
        }
    })
}
```

我们还应该在 `cmd/controller-manager/main中导入 <code>github.com/chaos-mesh/chaos-mesh/controllers/helloworldchaos` 。 o</code>然后它将在控制器启动时在路线表上注册。

## 制作Docker图像

对象已成功添加，您可以制作一个停靠图像：

```bash
制造业：
```

然后将其推到您的注册表：

```bash
制作码头
```

如果您的Kind部署了Kubernetes集群，您需要将图像加载到Kind：

```bash
类型加载Docker-image localhost:5000/pingcap/chaos-mesh:latest
get load docker-image localhost:5000/pingcap/chaos-daemon:最新的
ind load docker-image localhost:5000/pingcap/chaos-dashboard:latest
```

## 运行故障

你们几乎在那里。 在这个步骤中，您将拉取图像并将其应用于测试。

在您为Chaos Mesh 拉取任何图像之前(使用 `helm install` 或 `helm 升级`), 修改 [值。 aml](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/values.yaml) 头盔模板将默认图像替换为您刚才推送到本地注册表。

在这种情况下，模板使用 `pingcap/chaos-mesh:latest` 作为默认目标注册表。 所以您需要将其替换为环境变量 `DOCKER_REGISTRY`其值(默认 `localhost:5000`)，如下所示：

```yaml
群集缩放：true

# 也可查看 clusterScoped and controllerManager.serviceAccount
rbac:
  creat: true

controllerManager:
  serviceAccount: chaos-controller-manager
...
  image: localhost:5000/pingcap/chaos-mesh:latest
  ...
chaosDaemon:
  images localhost:5000/pingcap/chaos-daemon:latest
...
仪表盘：
  图像：localhost:5000/pingcap/chaos-ashboard：最新
...
```

现在采取以下步骤来运行混乱状态：

1. 创建命名空间 `chaos-测试`

   ```bash
   kubectl 创建命名空间chaos-测试
   ```

2. 获取Chaos Mesh的相关自定义资源类型：

   ```bash
   kubectl 应用 -f 清单/
   ```

   您可以看到CRD `helloworldchaos` 已被创建：

   ```log
   customcedefinition.apiextensions.k8s.io/helloworldchaos.chaos-mesh.org 已创建
   ```

   现在您可以使用下面的命令获取CRD：

   ```bash
   kubectl get crd helloworldchaos.chaos-mesh.org
   ```

3. Install Chaos Mesh:

   - 对于helm 3.X

     ```bash
     helm install chaos-mesh helm/chaos-mesh --namespace=chaos-testing --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock
     ```

   - 对于头盔 2.X

     ```bash
     helm install hem/chaos-mesh --name=chaos-mesh --namespace=chaos-test--set chaosDaemon.runtime=containerd --set chaemon.socketPath=/run/containerd/containerd.sock
     ```

   要验证您的安装，请从 `chaos-testing` namespace：

   ```bash
   kubectl 获取 pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
   ```

   > **注：**
   > 
   > 参数 `--set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock` 用于支持实物网络混乱。

4. 在下面的任何位置创建 `chaos.yaml`

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: HelloWorldChaos
   metadata:
     name: hello-world
     namespace: chaos-testation
   spec: {}
   ```

5. 应用chaos：

   ```bash
   kubectl 应用 -f / path/to/chaos.yaml
   ```

   ```bash
   kubectl get HelloWorldChaos -n chaos-testing
   ```

   现在你应该能够检查 `Hello 世界！` 结果是 `chaos-controller-manager`:

   ```bash
   kubectl logs chaos-controller-manager-{pod-post-fix} -n chaos-测试
   ```

   日志如下：

   ```log
   2020-09-07T09:21:29.301Z INFO controllers.HelloWorldChaos Hello World！    {"concilier": "helloworldchaos"}
   2020-09-07T09:21:29.308Z DEBUG controller-runtime.controller successed {"controller": "helloworldchaos", "request": "chaos-testing/hello-world"}
   ```

   > **注：**
   > 
   > `{pod-post-fix}` 是由 Kubernetes 生成的随机字符串。 你可以通过执行 `kubectl 获取pod -n chaos-tests` 来检查它。

## 今后的步骤

恭喜！ 您刚刚成功添加了Chaos Mesh的混乱类型。 让我们知道你在这个过程中是否遇到任何问题。 如果你觉得喜欢做其他类型的贡献，请参考 [将设施添加到混乱守护进程](add_chaos_daemon.md)。
