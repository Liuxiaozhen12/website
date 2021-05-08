---
id: iochaos_expert
title: IOChaos Experiment
sidebar_label: IOChaos Experiment
---

这个文档会让您穿越IOChaos实验。

IOChaos 允许您模拟文件系统错误，如IO 延迟和读/写错误。 当你的程序运行 IO 系统调用时，它可能会注入延迟和故障，如 `打开`， `读取`和 `写入`。

## 配置文件

下面是IOChaos的 YAML 文件样本：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: IoChaos
metdata:
  name: io-delayy-example
spec:
  action: 延迟
  mode: one
  selector:
    label Selector:
      app: etcd
  volumePath: /var/run/etcd
  path: '/var/run/etcd/**/*'
  delay: '100ms'
  percent: 50
  dur: '400s'
  scheduler:
    cron: '@每 10m'
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。

| 字段       | 描述                                                                                                                                   | 示例值                                                                                       |
|:-------- |:------------------------------------------------------------------------------------------------------------------------------------ |:----------------------------------------------------------------------------------------- |
| **模式**   | 定义选择器的模式。                                                                                                                            | `一个` / `所有` / `已修复` / `固定百分比` / `随机最大百分比`                                                 |
| **选择器**  | 指定IO chaos要注入的点数。                                                                                                                    |                                                                                           |
| **行动**   | 代表IOChaos动作。 更多详情请参阅 [IOChaos](#iavailable-actions-for-iochaos) 可用的操作。                                                               | `延迟` / `故障` / `景点覆盖`                                                                      |
| **音量路径** | 目标卷的挂载路径。                                                                                                                            | `"/var/run/etcd"`                                                                         |
| **延迟**   | 指定故障注入的延迟。 持续时间可能是一个字符串，其符号顺序为十进制数字，每个字符串都有一个可选的分数和一个单位后缀。 有效时间单位为“ns”、“us”（或“微粒”）、“ms”、“s”、“m”和“h”。                                 | `"300毫秒"` / `"2h45m"`                                                                     |
| **错误**   | 定义一个 IO 动作返回的错误代码。 查看 [常见的Linux系统错误](#common-linux-system-errors) 获取更多Linux系统错误代码。                                                   | `2`                                                                                       |
| **attr** | 定义要覆盖的属性和相应的值                                                                                                                        | [示例：](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples/io-attr-example.yaml) |
| **百分比**  | 定义注射错误的概率百分比值。                                                                                                                       | `100` (默认值)                                                                               |
| **路径**   | 定义注射IOChaos操作的文件路径。 它应该是你想要注入错误或延迟的文件的一个手机。 它是基于 [全球图案](https://www.man7.org/linux/man-pages/man7/glob.7.html) 的，应该在 volumePath 目录中。 | "/var/run/etcd/\*\*/\*"                                                             |
| **方法**   | 定义IOChaos注射IOChaos操作的IO方法。 它是一个字符串数组表示的。                                                                                             | `打开` / `读取` 查看 [可用方法](#available-methods) 了解更多详情。                                         |
| **持续时间** | 表示混乱动作的持续时间。 持续时间可能是一个字符串，其符号序列的十进制数字，每个字符串都有一个可选的分数和一个单位后缀。                                                                         | `"300毫秒"` / `"2h45m"`                                                                     |
| **调度器**  | 定义故障实验运行时间的调度规则。                                                                                                                     | see [robfig/cron](https://godoc.org/github.com/robfig/cron)                               |

## 用法

假设您正在使用 `示例/io-mixedexample.yaml`，您可以运行以下命令来创建一个chaos实验：

```bash
kubectl 应用 -f 示例/io-mixedexample.yaml
```

## IOChaos 可用的操作

IOChaos目前支持以下行动：

- **延迟**: IO 延迟操作。 您可以在IO 操作返回结果之前指定延迟。
- **故障**: IO 故障操作。 在此模式下，IO 操作返回一个错误。
- **吸引覆盖**: 覆盖文件的属性。

### 延迟

如果您正在使用 `延迟` 动作，您可以编辑下面的规格：

```yaml
示例：
  操作：延迟
  延迟：“1毫秒”
```

它会在选定的方法中注入1毫秒的延迟。

### 故障

如果您正在使用 `故障` 动作，您可以编辑下面的规格：

```yaml
示例：
  操作：故障
  错误: 32
```

选中的方法返回了32个错误，这意味着 `管道断了`。

### 景点覆盖

如果您正在使用 `个景点覆盖` 模式，您可以编辑下面的规格：

```yaml
示例：
  操作：吸引
  景点：
    权限：72
```

然后，所选文件的权限将被千分之八的110覆盖，这意味着文件无法读取或修改(没有 CAP_DAC_OVERRIDE)。 查看 [可用属性](#available-attributes) 以获取所有可能被覆盖的属性。

> **注意**:
> 
> 此属性可以通过 Linux 内核缓存，所以如果您的程序以前曾经访问过它，它可能没什么作用。

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

欲了解更多信息，请参阅 [相关的头文件](https://raw.githubusercontent.com/torvalds/linux/master/include/uapi/asm-generic/errno-base.h)。

## 可用的方法

现有方法如下：

- 查找
- 忘记了
- getattr
- setattr
- 读取链接
- mknod
- mkdir
- 取消链接
- rmdir
- symlink
- 重命名：
- 链接
- 打开
- 已读
- 写
- 刷入
- 发布
- fsync
- opendir
- 增益者
- releaseder
- fsyncdir
- 状态
- setxattr
- getxattr
- listxattr
- removexattr
- 访问
- 创建
- getlk
- setlk
- bmap

## 可用属性

可用属性及其含义在此列出：

- `ino`, 文件的 inode
- `大小`, 总大小, 单位为字节
- `块`, 已分配的 512B 块数
- `atime`, 最后一次访问的时间
- `mtime`, 最后修改时间
- `ctime`, 最后一次状态变化的时间
- `类型`, 文件类型。 它可以是 `namedPipe`, `charDevel`, `blockD设备`, `目录`, `regular File`, `符号链接` 或 `套接字`
- `perm`, 文件权限
- `nlink`, 硬链接数量
- `uid`, 所有者的用户id
- `gid`, 所有者的组 id
- `rdev`, 设备 ID (如果特殊文件)
