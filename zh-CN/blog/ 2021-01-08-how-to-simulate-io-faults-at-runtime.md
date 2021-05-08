---
slug: /how to simplate-io-defaults-at runtime
title: '如何模拟运行时的 I/O 故障'
author: Keao Yang
author_title: Chaos Mesh 维护者
author_url: https://github.com/YangKeao
author_image_url: https://avatars2.githubusercontent.com/u/5244316
image: /img/如何模拟运行时间的io-故障 jpg
tags:
  - Chaos Mesh
  - Chaos工程
  - 故障注入
---

![Chaos Engineering-如何模拟运行时的 I/O 故障](/img/how-to-simulate-io-faults-at-runtime.jpg)

在生产环境中，由于磁盘故障和管理员错误等各种事件，文件系统可能出现故障。 作为一个Chaos Engineering 平台，Chaos Mesh自文件系统最初版本以来一直支持在文件系统中模拟I/O 故障。 只需添加一个 IOChaos CustomResourceDefinition (CRD)，我们就可以观看文件系统如何失败并返回错误。

<!--truncate-->

然而，在Chaos Mesh 1.0之前，这项试验并不容易，可能耗费了大量资源。 我们需要通过突变接口web钩子注入侧边际容器并重写 `ENTRYPOINT` 命令。 即使没有注入过错，注入的侧翼集装箱也造成了大量的间接费用。

Chaos Mesh 1.0 改变了所有这一切。 现在，我们可以使用 IOChaos 在运行时将故障注入到文件系统。 这简化了程序，大大减少了系统间接费用。 这篇博文介绍了我们如何实现IOChaos实验而不使用侧边栏。

## I/O 故障输入

模拟运行时的 I/O 故障 我们需要在程序启动 [系统调用](https://man7.org/linux/man-pages/man2/syscall.2.html) (例如读取和写入) 后将故障注入文件系统，但在呼叫请求到达目标文件系统之前。 我们可以通过以下两种方式之一这样做：

- 使用Berkeley Packet 过滤器 (BPF)；然而它 [不能用于注入延迟](https://github.com/iovisor/bcc/issues/2336)。
- 在目标文件系统之前添加一个名为 ChaosFS 的文件系统层。 ChaosFS 使用目标文件系统作为后端，并接收操作系统的请求。 整个通话链接是 **目标程序系统扫描** -> **Linux 内核** -> **ChaosFS** -> **目标文件系统**。 由于ChaosFS可以定制，因此我们可以在我们想要的时候注入延迟和错误。 因此，ChaosFS是我们的选择。

但是，ChaosFS有几个问题：

- 如果ChaosFS 读取并写入目标文件系统中的文件， 我们需要 [挂载](https://man7.org/linux/man-pages/man2/mount.2.html) ChaosFS 到一个不同于Pod 配置中指定的目标路径的路径。 ChaosFS **cannot** be mounted to the path of the target directory.
- We need to mount ChaosFS **before** the target program starts running. 这是因为新挂载的 ChaosFS 仅对程序在目标文件系统中新打开的文件生效。
- 我们需要将 ChaosFS 挂载到目标画面 `mnt` 命名空间中。 欲了解详情，请参阅 [mount_namespaces(7) — Linux 手册页面](https://man7.org/linux/man-pages/man7/mount_namespaces.7.html)。

在Chaos Mesh 1.0之前，我们使用 [突变接纳webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) 实现IOChaos。 这种技术处理了上述三个问题，使我们能够：

- 在目标容器中运行脚本。 此操作更改了 ChaosFS 后端文件系统的目标目录 (例如) 从 `/mnt/a` 到 `/mnt/a_bak`以便我们可以把ChaosFS 挂载到目标路径(`/mnt/a`)。 修改启动Pod的命令。 例如，我们可以修改原来的命令 `/app` 到 `/waitfs.sh /app`。
- `等候.sh` 脚本一直在检查文件系统是否已成功挂载。 如果它已经挂载， `/app` 已经启动。
- 在 Pod 中添加一个新容器来运行 ChaosFS。 此容器需要与目标容器共享音量(例如， `/mnt`) 然后我们将此卷挂载到目标目录(例如， `/mnt/a`)。 我们还正确地启用了该卷的挂载 [挂载传播](https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation) 来穿透共享，然后穿透从子到目标。

这三种方法允许我们在程序运行时注入I/O 故障。 然而，注射很难做到：

- 我们只能将错误注入卷子目录，而不是注入整个卷。 此工作将替换 `mv` (重命名) 为 `挂载移动` 移动目标卷的挂载点。
- 我们必须在 Pod 中明确写入命令，而不是暗示使用图像命令。 否则，在文件系统挂载后， `/waitfs.sh` 脚本无法正确启动程序。
- 相应的容器需要有一个正确的配置来进行挂载传播。 Due to potential privacy and security issues, we **could not** modify the configuration via the mutating admission webhook.
- 注入配置有问题。 更糟糕的是，在配置能够注入错误之后，我们不得不创建一个新的 Pod 。
- 当程序正在运行时，我们不能撤回ChaosFS。 即使未注入故障或错误，性能也受到严重影响。

## 注入I/O 错误，不需要突变的接入web钩子

在没有变异的接入web钩子的情况下打碎这些困难的坚果怎么办？ 让我们回头思考一下我们为什么使用了 web 钩子来添加ChaosFS 运行的容器的原因。 我们这样做是为了将文件系统挂载到目标容器中。

事实上，还有另一种解决办法。 不要将容器添加到波德， 我们可以先使用 `设置` Linux 系统调用来修改当前进程的命名空间，然后使用 `挂载` 调用来挂载ChaosFS 到目标容器。 假设要注入的文件系统是 `/mnt`。 新的注入过程如下：

1. 使用 `设置` 作为当前进程进入目标容器的 mnt 命名空间。
2. 执行 `mount --move` 移动 `/mnt` 到 `/mnt_bak`
3. 挂载ChaosFS 到 `/mnt` 并使用 `/mnt_bak` 作为后端。

过程结束后，目标容器将打开、读取并通过ChaosFS写入 `/mnt` 文件。 这样就更容易注入延迟或故障。 然而，仍有两个问题需要回答：

- 您如何处理目标进程已经打开的文件？
- 鉴于打开文件时我们无法卸载文件系统，您如何恢复进程？

### 动态替换文件描述符

**ptract 解决了上面两个问题。** 我们可以在运行时使用 ptract 替换打开的文件描述符，并替换当前工作目录(CWD) 和mmap。

#### 使用 ptrace 允许追踪运行一个二进制程序

[ptrace](https://man7.org/linux/man-pages/man2/ptrace.2.html) 是一个强大的工具，可以使目标进程 (追踪) 运行任何系统调用或二进制程序。 为了跟踪程序运行， ptract 修改了RIP-标出的地址到目标进程，并添加了一个 `int3` 指令来触发一个断点。 当二进制程序停止时，我们需要恢复注册和记忆。

> **注：**
> 
> 在 [x86_64 架构](https://en.wikipedia.org/wiki/X86_assembly_language), RIP注册表(也称为指令指针)总是指向运行下一个指令的内存地址。 要将程序加载到目标进程内存空间：

1. 在目标程序中使用 ptract 调用mmap 来分配所需的内存。
2. 将二进制程序写入新分配的内存，并使RIP注册表指向它。
3. 在二进制程序停止后，通话快照来清理内存部分。

作为最佳做法， 我们常用 `进程_vm_writev` 替换ptract `POKE_TEXT` 写入，因为如果有大量数据可写， `process_vm_writev` 更有效地执行操作。

我们能够利用拖拉机来取代自己的直接外资。 现在我们只需要一种方法来实现这种替换。 此方法是 `dup2` 系统调用。

#### 使用 `dup2` 替换文件描述符

`dup2` 函数的签名是 `int dup2 (int oldfd, int newfd)；` 它用于创建旧的 FD 副本(`旧的`)。 此副本有一个 `newfd` 的FD 数。 如果 `新的` 已经对应于已打开文件的 FD 文件，那么已经打开的文件上的 FD 将自动关闭。

例如，当前进程打开 `/var/run/__chaosfs__test__/a` FD 是 `1`。 要将此打开的文件替换为 `/var/run/test/a`, 此进程执行以下操作：

1. 使用 `fcntl` 系统调用来获取 `Offlags` ( `打开` 系统调用的参数， 例如 `O_WRONLY` `/var/run/__chaosfs__test__/a`
2. 使用 `Iseet` 系统调用来获取 `寻找的当前位置`。
3. 使用 `打开` 系统调用来打开 `/var/run/test/a` 使用相同的 `关闭` 假定FD是 `2`。
4. Uses `Iseek` to change the `seek` location of the newly opened FD `2`.
5. Uses `dup2(2, 1)` to replace the FD `1` of `/var/run/__chaosfs__test__/a` with the newly opened FD `2`.
6. 关闭 FD `2`。

进程结束后，FD `1` 当前进程指向 `/var/run/test/a`。 为了让我们能够注入错误，目标文件上随后的任何操作都会穿过用户空间中的 [文件系统](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) (FUSE)。 FUSE 是Unix 和 Unix式计算机操作系统的一个软件接口，让非特权用户在不编辑内核代码的情况下创建自己的文件系统。

#### 写一个程序来使目标进程替换它自己的文件描述符

ptrack 和 dup2 的综合功能使得追踪器能够自行取代打开的FD。 现在，我们需要写一个二进制程序并运行它：

> **注：**
> 
> 在上述执行过程中，我们假定：
> 
> - 目标进程线程为 POSX 线程并共享打开的文件。
> - 当目标进程使用 `克隆` 函数创建线程时， `CLONE_FILES` 参数被传递。
> 
> 因此，Chaos Mesh只能在线程组中取代第一线程的FD。

1. 根据上面两节以及系统扫描指令的使用写入一个组装代码。 [这里](https://github.com/chaos-mesh/toda/blob/1d73871d8ab72b8d1eace55f5222b01957193531/src/replacer/fd_replacer.rs#L133) 是组装代码的一个例子。
2. 使用集成程序将代码转换为二进制程序。 我们使用 [dynasm-rs](https://github.com/CensoredUsername/dynasm-rs) 作为装配器。
3. 使用 ptract 来使目标进程运行此程序。 当程序运行时，FD将在运行时被替换。

### 全面故障注入进程

下图说明了I/O 故障注入过程：

![故障注入进程](/img/fault-injection-process.jpg)

<div style={{ margin: '1rem 0', fontStyle: 'italic', textAlign: 'center' }}> 故障注入进程 </div>

在这个图表中，每条水平线对应沿箭头运行的线程。 **挂载/Umount 文件系统** and **替换FD** 任务是按顺序仔细排列的。 鉴于上述程序，这种安排很有道理。

## 下一步

我讨论了如何实现故障注入以模拟运行时的 I/O 故障(见 [chaos-mesh/toda](https://github.com/chaos-mesh/toda))。 然而，目前的执行情况远非完美无缺：

- 不支持生成号码。
- 不支持 ioctl。
- Chaos Mesh 无法立即确定文件系统是否已成功挂载。 它只是在一秒钟之后才这样做。

如果您对Chaos Mesh感兴趣并想帮助我们改进它， 欢迎您加入 [我们的 Slack 频道](https://slack.cncf.io/) 或向我们 [GitHub 仓库提交您的合并请求或问题](https://github.com/chaos-mesh/chaos-mesh)。

这是Chaos Mesh实施系列中的第一个职位。 如果您想看到如何实现其他类型的故障注入，请保持敬业。
