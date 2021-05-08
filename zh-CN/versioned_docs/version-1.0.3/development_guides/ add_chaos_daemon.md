---
id: 添加设施至chaos_daemon
title: 添加设施到Chaos 守护进程
sidebar_label: 添加设施到Chaos 守护进程
---

在 [开发一个新的chaos](dev_hello_world.md), 我们添加了一个新的chaos 类型，名为 `HelloWorldChaos`, 将打印 `hello world` in `chaos-controller-manager` 要实际运行混乱， 我们需要配置Chaos Daemon 的一些设施，以便 `控制器管理器` 可以根据chaos 配置选择指定的 Pods，并将chaos请求发送给 `chaos-daemon` 对应于这些Pod。 完成后， `chaos-daemon` 可以最终运行chaos。

本指南涵盖以下步骤：

- [为 HelloWorldChaos 添加选择器](#add-selector-for-helloworldChaos)
- [实现 gRPC 接口](#implement-grpc-interface-for-chaos-request)
- [验证您的chaos](#verify-your-chaos)

## 为 HelloWorldChaos 添加选择器

在Chaos Mesh中，我们定义了 `spec.selecutor` 字段，以指定按命名空间、标签、批注等设置的chaos范围。 您可以参考 [定义Chaos 实验范围](../user_guides/experiment_scope.md) 以获取更多信息。 要指定 `HelloWorld` chaos：

1. 在 `HelloWorldChaos` 中添加 `Spec` 字段:

   ```go
   // HelloWorldChaos 是 helloworldchaos API
   类型的 HelloWorldChaos 结构。
       metav1。 ypeMeta `json:",inline"
       metav1。 bjectMeta `json:"元数据。 mitempty"

       // Spec 定义了pod chaos expert 的行为
       Spec HelloWorldSpec `json:"spec"
   }

   类型 HelloWorldSpec structor * *
       Selector SelectorSpec `json:"selector"
   }

   // GetSelector 是 Selector (for implementing SelectSpec)
   func (in *HelloWorldSpec) GetSelector() SelectorSpec {
       return in.Selector
   }
   ```

2. 生成 `spec` 字段的锅板功能。 这需要整合Chaos Mesh中的资源。

   ```bash
   生成
   ```

## 实现 gRPC 接口

为了 `chaos-daemon` 接受来自 `chaos-controller-manager`, `chaos-controller-manager` 和 `chaos-daemon` 需要一个新的 gRPC 接口。 采取下面的步骤添加 gRPC 接口：

1. 在 [chaosdaemon.proto](https://github.com/chaos-mesh/chaos-mesh/blob/master/pkg/chaosdaemon/pb/chaosdaemon.proto) 中添加 RPC 。

   ```proto
   service chaosDaemon {
       ...

       rpc ExecHelloWorldChaos(ExecHelloWorldRequest) 返回 (google.protobuf.Empty) {}
   }

   消息ExecHelloWorldRequest format@@
       string container_id = 1;
}
   ```

   您将需要更新此proto 文件生成的高黄金代码：

   ```bash
   制作proto
   ```

2. 在 `chaos-daemon` 中实现 gRPC 服务。

   在 [chaosdaemon](https://github.com/chaos-mesh/chaos-mesh/tree/master/pkg/chaosdaemon)下添加一个名为 `helloworld_server.go` 的新文件，其内容如下：

   ```go
   package chaosdaemon

   import (
       "context"
       "fmt"

       "github.com/golang/protobuf/ptypes/empty"

       "github.com/chaos-mesh/chaos-mesh/pkg/bpm"
       pb "github.com/chaos-mesh/chaos-mesh/pkg/chaosdaemon/pb"
   )

   func (s *daemonServer) ExecHelloWorldChaos(ctx context.Context, req *pb.ExecHelloWorldRequest) (*empty.Empty, error) {
       log.Info("ExecHelloWorldChaos", "request", req)

       pid, err := s.crClient.GetPidFromContainerID(ctx, req.ContainerId)
       if err != nil {
           return nil, err
       }

       cmd := bpm.DefaultProcessBuilder("sh", "-c", fmt.Sprintf("echo 'hello' `hostname`")).
           SetNS(pid, bpm.UtsNS).
           SetContext(ctx).
           Build()
       out, err := cmd. utput()
       如果是err ！ nil {
           return nil, err
       }
       如果镜头(出) ！ 0 ●
           日志。 nfo("cmd output", "output", string(out))
       }

       return &vain. mpty{}, nil
}
   ```

   `chaos-daemon` 收到 `ExecHelloWorldChaos` 请求后， `chaos-daemon` 将打印 `hello` 到这个容器的主机名。

3. 以调节方式发送 gRPC 请求。

   当CRD对象更新时(例如：创建或删除)，我们需要将对象中指定的状态与实际状态进行比较。 然后执行操作使群集的实际状态反映指定的状态。 这个进程叫做 `调节`。

   对于 `HelloworldChaos`, `chaos-controller-manager` 需要将chaos requests to `chaos-daemon` in `confirmation` 要做到这一点，我们需要更新文件 `控制器/helloworldchaos/types.go` 创建于 [开发一个新Chaos](./dev_hello_world.md) 的内容，内容如下：

   ```go
   package helloworldchaos

   import (
       "context"
       "errors"
       "fmt"

       "k8s.io/apimachinery/pkg/runtime"
       ctrl "sigs.k8s.io/controller-runtime"

       "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
       "github.com/chaos-mesh/chaos-mesh/controllers/common"
       pb "github.com/chaos-mesh/chaos-mesh/pkg/chaosdaemon/pb"
       "github.com/chaos-mesh/chaos-mesh/pkg/router"
       ctx "github.com/chaos-mesh/chaos-mesh/pkg/router/context"
       end "github.com/chaos-mesh/chaos-mesh/pkg/router/endpoint"
       "github.com/chaos-mesh/chaos-mesh/pkg/utils"
   )

   // endpoint is dns-chaos reconciler
   type endpoint struct {
       ctx.Context
   }

   // Apply applies helloworld chaos
   func (r *endpoint) Apply(ctx context.Context, req ctrl.Request, chaos v1alpha1.InnerObject) error {
       r.Log.Info("Apply helloworld chaos")
       helloworldchaos, ok := chaos.(*v1alpha1.HelloWorldChaos)
       if !ok {
           return errors.New("chaos is not helloworldchaos")
       }

       pods, err := utils.SelectPods(ctx, r.Client, r.Reader, helloworldchaos.Spec.GetSelector())
       if err != nil {
           return err
       }

       for _, pod := range pods {
           daemonClient, err := utils.NewChaosDaemonClient(ctx, r.Client, &pod, common.ControllerCfg.ChaosDaemonPort)
           if err != nil {
               r.Log.Error(err, "get chaos daemon client")
               return err
           }
           defer daemonClient.Close()
           if len(pod.Status.ContainerStatuses) == 0 {
               return fmt.Errorf("%s %s can't get the state of container", pod.Namespace, pod.Name)
           }

           containerID := pod.Status.ContainerStatuses[0].ContainerID

           _, err = daemonClient.ExecHelloWorldChaos(ctx, &pb.ExecHelloWorldRequest{
               ContainerId: containerID,
           })
           if err != nil {
               return err
           }
       }

       return nil
   }

   // Recover means the reconciler recovers the chaos action
   func (r *endpoint) Recover(ctx context.Context, req ctrl.Request, chaos v1alpha1.InnerObject) error {
       return nil
   }

   // Object would return the instance of chaos
   func (r *endpoint) Object() v1alpha1.InnerObject {
       return &v1alpha1.HelloWorldChaos{}
   }

   func init() {
       router.Register("helloworldchaos", &v1alpha1.HelloWorldChaos{}, func(obj runtime.Object) bool {
           return true
       }, func(ctx ctx.Context) end.Endpoint {
           return &endpoint{
               Context: ctx,
           }
       })
   }
   ```

   > **注：**
   > 
   > 在我们的情况下， `Recover` 功能没有什么作用，因为 `HelloWorldChaos` 只打印了一些日志，不会改变任何东西。 您可能需要在开发过程中实现 `恢复` 功能。

## 验证您的chaos

现在您都已设置。 现在是验证您刚刚创建的混乱类型的时候了。 采取以下步骤：

1. 制作停靠镜像。 参考 [使Docker镜像](dev_hello_world.md#make-the-docker-image)。

2. 升级Chaos Mesh. 既然我们已经在 [开发新Chaos](dev_hello_world.md#run-chaos)中安装了Chaos Mesh，我们只需要重新启动它，使用最新的图像：

   ```bash
   kubectl rollout restart deployment chaos-controller-manager -n chaos-testing
   kubectl rollout restart daemonset chaos-daemon -n chaos-testing
   ```

3. 部署Pods进行测试：

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/chaos-mesh/apps/master/ping/busybox-statefulset.yaml
   ```

   此命令在 `busybox` 命名空间中部署两个Pods。

4. 创建chaos YAML 文件：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: HelloWorldChaos
   metadata:
     name: busybox-helloworld-chaos
   spec:
     selector:
       namespace:
         - busybox
   ```

5. 应用chaos：

   ```bash
   kubectl 应用 -f / path/to/helloworld.yaml
   ```

6. 验证您的混乱。 有不同的日志来检查您的聊天室是否如预期那样起作用：

   - 检查 `chaos-controller-manager` 的日志：

     ```bash
     kubectl logs chaos-controller-manager-{pod-post-fix} -n chaos-测试
     ```

     日志如下：

     ```log
     2020-09-09T09:13:36.018Z INFO controllers.HelloWorldChaos Reconciling helloworld chaos {"concilier": "helloworldchaos"}
     2020-09-09T09:13:36.018Z INFO controllers.HelloWorldChaos apply helloworld chaos {"concilier": "helloworldchaos"}
     ```

   - 检查 `chaos-daemon` 的日志：

     ```bash
     kubectl logs chaos-daemon-{pod-post-fix} -n chaos-testing
     ```

     日志如下：

     ```log
     2020-09-09T09:13:36。 36Z INFO chaos-daemon-server exec hello world chaos {"request": "container_id:\"docker:/8f2918ee05ed587f7074a923cede3bbe5886277fa95d989e513f7b7e831da5\"}
     2020-09-09T09T09:13:36. 44Z INFO chaos-daemon-server building 命令 {"command": "nsenter -u/proc/4564/ns/uts -- sh -c echo 'hello' `hostname`}
     2020-09-09T09:13:36. 58Z INFO chaos-daemon-server cmd output {"output": "hello busybox-1\n"}
     2020-09-09T09:13:36. 64Z INFO chaos-daemon-server exec hello world chaos {"request": "container_id:\"docker://53e982ba5593fa87648edba665ba0f7da3f58df67f8b70a1354ca00447c00524\"}
     2020-09-09T09T09:13:36。 66Z INFO chaos-daemon-server building 命令 {"command": "nsenter -u/proc/45620/ns/uts -- sh -c echo 'hello' `hostname`"}
     2020-09-09T09:13:36. 70Z INFO chaos-daemon-server cmd output {"output": "hello busybox-0\n"}
     ```

     We can see the `chaos-daemon` prints `hello` to these two Pods.
