---
id: develop_a_new_chaos
title: 开发一个新的Chaos
sidebar_label: 开发一个新的Chaos
---

[准备开发环境](setup_env.md)后，让我们开发一种新型的混乱，HelloWorldChaos，它只能打印一个“你好的世界！ 消息到日志。 一般来说，要为Chaos Mesh添加一个新的混乱类型，您需要采取以下步骤：

1. [在控制器中添加chaos对象](#add-the-chaos-object-in-controller)
2. [注册CRD](#register-the-crd)
3. [实现架构类型](#implement-the-schema-type)
4. [制作Docker图像](#make-the-docker-image)
5. [运行故障](#run-chaos)

## 在控制器中添加chaos对象

在Chaos Mesh中，所有的混乱类型都由控制器经理管理。 要添加一个新的混乱类型，您需要按照以下步骤的指示，从在控制器中添加相应的调节器类型开始：

1. 在控制器管理器 [main.go](https://github.com/chaos-mesh/chaos-mesh/blob/master/cmd/controller-manager/main.go#L104) 中添加 HelloWorldChaos 对象。

   您将注意到现有的混乱类型，如PodChaos、NetworkChaos和IOChaos。 在下面添加新类型：

   ```go
    如果是err = (&controllers.HelloWorldChaosReconciler@un.org
        Client: mgr. etClient(),
        Log: ctrl.Log.Withname("Controllers").WidName("HelloWorldChaos"),
    })。 etupWidManager(mgr)；err != nil 然后
        setuplog。 rror(err, "无法创建控制器", "控制器", "HelloWorldChaos")
        os.Exit(1)
}
   ```

2. 在 [控制器](https://github.com/chaos-mesh/chaos-mesh/tree/master/controllers)之下，创建一个 `Helloworldchaos_controller.go` 文件并编辑如下：

   ```go
   软件包控制器

   导入 (
     "github.com/go-logr/logr"

     chaosmeshv1alpha1 "github. om/chaos-mesh/chaos-mesh/api/v1alpha1"

     ctrl "sigs.k8s.io/controller-runtime"
     "sigs.k8s. o/controller-runtime/pkg/client”
   ()

   // HelloWorldChaosReconciler调节HelloWorldChaos object
   type HelloWorldChaosReconciler struct ow.
     client. lient
     Loogr.Logger
   }

   // +kubebuilder:rbac:groups=chaos-mesh. rg,resources=helloworldchaos,verbs=get;list;watch;create;update;patch;delete
   // +kubebuilder:rbac:groups=chaos-mesh.org,resources=helloworldchaos/status,verbs=get;update;patch

   func (r *HelloWorldChaosReconciler) Reconcil(req ctrl)。 赤道) (ctrl.Result, error) 电子邮件：
     Logger := r.Log。 ithValues("调节器", "helloworldchaos")

     // `HelloWorldChaos`的主要逻辑，它打印了一个log `Hello World!` 并没有返回任何东西。
     logger.Info("Hello World!")

     返回 ctrl.Result{}, nil
   }

   func (r *HelloWorldChaosReconterer) SetupWidManager(mgr ctrl). 相应的错误。您需要
   // 导出`HelloWorldChaos` 对象，代表用户适用的 yaml schema 内容。
   返回 ctrl.NewControllerManagedBy(mgr)。
     For(&chaosmeshv1alpha1.HelloWorldChaos{})。
     Complete(r)
   }
   ```

> **注：**
> 
> 评论 `// +kubebuilder:rbac:groups=chaos-mesh.org...` 是一个决定哪个账户可以访问此调节器的权威控制机制。 要让仪表板和chaos-controller-manager能够访问它，您需要相应修改 [Controller-manager-rbac.yaml](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/templates/controller-manager-rbac.yaml)：

```yaml
- apiGroups: ['chaos-mesh. rg']
  资源：
    - podchaos
    - networkchaos
    - iochaos
    - helloworldchaos # 在所有chaos-mesh中添加此行。 rg 组
  动词: ['*' ]
```

## 注册CRD

HelloWorldChaos对象是Kubernetes的自定义资源对象。 这意味着您需要在 Kubernetes API中注册相应的 CRD。 为了做到这一点，修改 [kustomization.yaml](https://github.com/chaos-mesh/chaos-mesh/blob/master/config/crd/kustomization.yaml) 添加相应的行如下所示：

```yaml
资源：
  - bases/chaos-mesh.org_podchaos.yaml
  - bases/chaos-mesh.org_networkchaos.yaml
  - bases/chaos-mesh.org_iochaos.yaml
  - bases/chaos-mesh.org_helloworldchaos.yaml # 这是新行
```

## 实现架构类型

要实现新的 chaos 对象的架构类型，在 [api 目录](https://github.com/chaos-mesh/chaos-mesh/tree/master/api/v1alpha1) 中添加 `helloworldchaos_types.go` 并修改如下：

```go
package v1alpha1

import (
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// +kubebuilder:object:root=true

// HelloWorldChaos is the Schema for the helloworldchaos API
type HelloWorldChaos struct {
    metav1.TypeMeta   `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`
}

// +kubebuilder:object:root=true

// HelloWorldChaosList contains a list of HelloWorldChaos
type HelloWorldChaosList struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ListMeta `json:"metadata,omitempty"`
    Items           []HelloWorldChaos `json:"items"`
}

func init() {
    SchemeBuilder.Register(&HelloWorldChaos{}, &HelloWorldChaosList{})
}
```

添加此文件后，HelloWorldChaos schema类型已被定义，并可以被以下YAML行调用：

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: HelloWorldChaos
metadata:
  namespace: <name-of-this-resource>
  namespace: <ns-of-this-resource>
```

## 制作Docker图像

对象已成功添加，您可以制作一个停靠镜像并将它推送到您的注册表：

```bash
做
让码头
```

> **注：**
> 
> 默认 `DOCKER_REGISTRY` 是 `localhost:5000`, 它预设在 `hack/kind-cluster-build.sh`。 您可以将它覆盖到您有访问权限的注册表。

## 运行故障

你们几乎在那里。 在这个步骤中，您将拉取图像并将其应用于测试。

在您为Chaos Mesh 拉取任何图像之前(使用 `helm install` 或 `helm 升级`), 修改 [值。 aml](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/values.yaml) 头盔模板，用您刚才推送到本地注册表的方式替换默认图像。

在这种情况下，模板使用 `pingcap/chaos-mesh:latest` 作为默认目标注册表。 所以您需要将其替换为 `localhost:5000`，如下所示：

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

1. 获取Chaos Mesh的相关自定义资源类型：

   ```bash
   kubectl 应用 -f 清单/
   kubectl 获取 crd podchaos.chaos-mesh.org
   ```

2. Install Chaos Mesh:

   ```bash
   helm install helm/chaos-mesh --namespace=chaos-test--set chaosDaemon.runtime=containerd --set chaemon.socketPath=/run/containerd/containerd.sock
   kubectl get pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
   ```

   参数 `--set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock` 用于支持实物网络混乱。

3. 在下面的任何位置创建 `chaos.yaml`

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: HelloWorldChaos
   metadata:
     name: hello-world
     namespace: chaos-testing
   ```

4. 应用chaos：

   ```bash
   kubectl apply -f /path/to/chaos.yaml
   kubectl get HelloWorldChaos -n chaos-测试
   ```

   现在你应该能够检查 `Hello World！` 结果导致日志：

   ```bash
   kubectl logs chaos-controller-manager-{pod-post-fix} -n chaos-测试
   ```

   > **注：**
   > 
   > `{pod-post-fix}` 是由 Kubernetes 生成的一个随机字符串，您可以通过执行 `kubectl 获取po -n chaos-treatment` 来检查它。

## 今后的步骤

恭喜！ 您刚刚成功添加了Chaos Mesh的混乱类型。 让我们知道你在这个过程中是否遇到任何问题。 如果你喜欢做其他类型的贡献, 请参阅将设施添加到chaos daemon (WIP)。
