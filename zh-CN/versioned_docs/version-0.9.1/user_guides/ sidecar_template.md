---
id: sidecar_template
title: Sidecar Template
sidebar_label: Sidecar Template
---

The following content is the common template ConfigMap defined for injecting IOChaos sidecar, you can also find this example [here](https://github.com/chaos-mesh/chaos-mesh/blob/release-0.9/manifests/chaosfs-sidecar.yaml):

## 模板配置地图

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: chaosfs-sidecar
  labels:
    app.kubernetes.io/component: template
data:
  data: |
    initContainers:
    - name: inject-scripts
      image: pingcap/chaos-scripts:latest
      imagePullPolicy: Always
      command: ["sh", "-c", "/scripts/init.sh -d {{.DataPath}} -f {{.MountPath}}/fuse-data"]
    containers:
    - name: chaosfs
      image: pingcap/chaos-fs:latest
      imagePullPolicy: Always
      ports:
      - containerPort: 65533
      securityContext:
        privileged: true
      command:
        - /usr/local/bin/chaosfs
        - -addr=:65533
        - -pidfile=/tmp/fuse/pid
        - -original={{.MountPath}}/fuse-data
        - -mountpoint={{.DataPath}}
      volumeMounts:
        - name: {{.VolumeName}}
          mountPath: {{.MountPath}}
          mountPropagation: Bidirectional
    volumeMounts:
    - name: {{.VolumeName}}
      mountPath: {{.MountPath}}
      mountPropagation: HostToContainer
    - name: scripts
      mountPath: /tmp/scripts
    - name: fuse
      mountPath: /tmp/fuse
    volumes:
    - name: scripts
      emptyDir: {}
    - name: fuse
      emptyDir: {}
    postStart:
      {{.ContainerName}}:
        command:
          - /tmp/scripts/wait-fuse.sh
```

模板配置由 [转到模板](https://golang.org/pkg/text/template/) 机制定义了一些变量。 这个示例有四个参数：

- 数据路径：原始数据目录
- 挂载路径：注入chaosfs sidecar之后，数据目录将被挂载到 {{.MountPath}}/fuse-data
- 卷名称：诗人使用的数据卷名称
- 容器名称：侧边栏注入到哪个容器

对于此模板中定义的字段，我们有以下一些简短的描述：

- **initContainers**: 定义 [initContainer](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) 需要注入.
- **容器**: 定义需要注入的侧ecar 容器。
- **volumeMounts**: 定义新的volumeMounts 或覆盖目标坑中每个容器的旧卷Mounts
- **音量**: 定义目标点的新量或覆盖目标点中的旧量。
- **poststart**: 先创建一个容器后调用。 如果处理程序失败，容器将失败。

> **注：**
> 
> Chaos controller-manager only watches template config map with label selector 指定 `--template-labels`, 默认情况下，此标签 是 `应用。 ubernetes.io/component=template` 如果你的 Chaos Mesh 是由头盔部署的。
> 
> 每个模板配置地图都应该部署在与Chaos Mesh相同的命名空间中。 并且它是通过配置地图的名称来识别的，在上述例子中，它是 `chaosfs-sidecar`。
> 
> 模板配置内容应该在 `数据` 字段中。 这意味着无法在一个配置映射中定义两个模板，您必须使用下面的示例这样的配置映射。

```yaml
---
apiVersion: v1
kind: ConfigMap
metdate:
  name: chaosfs-sidecar0
  标签:
    app.kubernetes。 o/component: template
data:
  data: |
    xxxx

---
apiVersion: v1
kind: ConfigMap
metdate:
  name: chaosfs-sidecar1
  labels:
    app. ubernetes.io/component：模板
数据：
  数据：|
    xxxx
```

### 容器

#### `chaosfs`

`chaosfs` 容器被设计为侧边容器。 [chaosfs](https://github.com/chaos-mesh/chaos-mesh/tree/master/cmd/chaosfs) 程序运行在此容器中。

`chaosfs` 使用 [fuser libary](https://github.com/hanwen/go-fuse) 和 [fusermount](https://www.kernel.org/doc/Documentation/filesystems/fuse.txt) 工具来实现fuse-daemon 服务并挂载应用程序的数据目录。 `chaosfs` 劫持应用程序的所有文件系统IO 动作，因此它可以用于模拟各种真实世界IO 错误。

以下配置会向目标点注入 `chaosfs` 容器，并在此容器中启动 `chaosfs` 流程。

此外， `chaosfs` 容器应该按照 `的特权` 和 [`挂载`](https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation) 字段在 `chaosfs` Container运行。 卷积应设置为 `双向`。

`chaosfs` 使用 `fusermount` 挂载应用程序容器的数据目录 `chaosfs` 容器。

如果任何带有 `的pod双向` 挂载传播到同一卷上的任何东西。 带有 `HostToContainer 的容器` 挂载传播将看到它。

此模式等于 [Linux 内核文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt) 中描述的 `rslos` mount propage.

更多关于 `挂载传播` 的详细信息可以在这里 [](https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation) 找到。

```yaml
containers:
  - name: chaosfs
    image: pingcap/chaos-fs
    imagePullpolicy: Always
    ports:
      - containerPort: 65534
    securityContext:
      privileged: true
    command:
      - /usr/local/bin/chaosfs
      - -addr=:65534
      - -pidfile=/tmp/fuse/pid
      - -original=/var/lib/tikv/fuse-data
      - -mountpoint=/var/lib/tikv/data
    volumeMounts:
      - name: tikv
        mountPath: /var/lib/tikv
        mountPropagation: Bidirectional
```

`chaosfs` 的描述：

- **添加**: 定义Grpc 服务器的地址，默认值：“:65534”。
- **pidfile**: 定义用于记录 `chaosfs 的 pid` 进程的 pid 文件。
- **原版**: 定义引信目录。 此目录通常设置为与应用程序数据目录相同的级目录。
- **挂载点**: 定义挂载原始目录的挂载点。

此值应设置为目标应用程序的数据目录。

#### `chaos-scripts`

`chaos-script` 容器用于将一些脚本注入到目标pods，包括 [等待引信.sh](https://github.com/chaos-mesh/chaos-mesh/blob/master/scripts/wait-fuse.sh)

`等待引信.sh` 被应用程序容器使用，以确保应用程序启动前的fuse-daemon 服务器正常运行。

`chaos-script` 通常被用作一个 initContainer 来做一些制作。

以下配置使用 `chaos-script` 容器注入脚本，并将脚本移动到 `/tmp/脚本` 目录使用 `init.sh` `/tmp/脚本` 是一个 [空卷](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) 用于与所有容器共享脚本。

所以您可以在 tikv 容器中使用 `等候引信.sh` 脚本来确保在应用程序启动前Fuse-daemon 服务器正常运行。

此外， `init. h` 在 [PersistentVolumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) tikv 的目录创建一个名为 `fuse-data` 的目录，作为Fuse-daemon 服务器的原目录和原目录是必需的。

您还应该在应用程序的PersistentVolumes目录中创建原始目录。

```yaml
initContainers：
  - 名称：注入脚本
    image: pingcap/chaos-scripts:latest
    imagePullpolicy: 总是
    命令: ['sh', '-c', '/scripts/init.sh -d /var/lib/tikv/data -f /var/lib/tikv/fuse-data']
```

`init.sh` 的用法：

```bash
$./scripts/init.sh -h
```

预期输出：

```bash
USAGE: ./scripts/init. h [-d 数据目录] [-f fuse 目录]
用于做一些准备
选项：
   -h 显示此消息
   -d <data directory>     应用程序的数据目录
   -f <fuse directory>     引信原始目录的数据目录
   -s <scripts directory>  脚本目录
EXAMPLES：
   init. h -d /var/lib/tikv/data -f /var/lib/tikv/fuse-data
```

### 提示

1. 用于定义数据目录的应用程序Container.volumeMounts应该设置为 `HostToContainer`。
2. `脚本` 和 `引信` 应创建空迪尔，并应挂载到土豆的所有容器。
3. 应用程序使用 `等候-fuse.sh` 脚本来确保fuse-daemon 服务器正常运行。

```yaml
帖子开始：
  tikv：
    命令：
      - /tmp/scripts/wait-fuse.sh
```

`正在等待。sh`:

```bash
$./script/等待fuse.sh -h
```

预期输出：

```bash
./script/wait-fuse.sh：选项需要参数 -- h
USAGE：./script/wait-fuse。 h [-a <host>][-p <port>]
等待引信服务器准备好
选项：
   -h 显示此消息
   -f <host>            设置目标文件
   -d <delay>           设置延迟时间
   -r <retry>           设置重试计数
EXAMPLES：
   等待安装。 h -f /tmp/fuse/pid -d 5 -r 60
```
