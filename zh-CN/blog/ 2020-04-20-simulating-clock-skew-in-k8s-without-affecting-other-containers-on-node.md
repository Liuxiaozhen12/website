---
slug: /模拟时钟片段中的 k8s-不影响其他容器的节点
title: 模拟不影响节点上其他容器的 K8 中时钟片段
author: Cwen Yin
author_title: Chaos Mesh 维护者
author_url: https://github.com/cwen0
author_image_url: https://avatars1.githubusercontent.com/u/22956341?v=4
image: /img/时钟-chaos-engineering-k8s.jpg
tags:
  - Chaos Mesh
  - Chaos工程
  - Kubernetes
  - 分布式系统
---

![分布式系统中的时钟同步](/img/clock-sync-chaos-engineering-k8s.jpg)

[Chaos MeshTM](https://github.com/chaos-mesh/chaos-mesh), 一个易于使用的Kubernetes (K8s)开源源码和原生的chaos工程平台， 有一个新功能，TimeChaos模拟 [时钟偏差](https://en.wikipedia.org/wiki/Clock_skew#On_a_network) 现象。 通常，当我们修改容器中的时钟时，我们想要一个 [最小的爆破半径](https://learning.oreilly.com/library/view/chaos-engineering/9781491988459/ch07.html)， 并且我们不希望更改会影响节点上的其他容器。 然而，在现实中，执行这项工作可能比你想象的更加困难。 Chaos Mesh 如何解决这个问题？

<!--truncate-->

在这个帖子中 我将描述我们如何通过不同的时钟扭曲方法来打破，以及Chaos Mesh中的TimeChaos如何使时间能够在容器中自由回旋。

## 模拟时钟扭曲而不影响节点上的其他容器

时钟偏差是指网络内节点上时钟之间的时间差异。 它可能在分布式系统中造成可靠性问题，这是复杂分布式系统的设计者和开发者所关心的。 例如，在分布式SQL数据库中。 保持同步的本地时钟跨节点以实现连贯的全局快照并确保交易的 ACID 属性是至关重要的。

目前有公认的 [个解决方案来同步时钟](https://pingcap.com/blog/Time-in-Distributed-Systems/), 但如果没有适当的测试，你永远不能确定你的实现是可靠的。

然后我们如何测试分布式系统中的全球快照一致性？ 答案很明显：我们可以模拟时钟扭曲，测试分布式系统是否能够在不正常的时钟条件下保持连贯的全球快照。 一些测试工具支持在容器中模拟时钟扭曲，但对物理节点有影响。

[TimeChaos](https://github.com/chaos-mesh/chaos-mesh/wiki/Time-Chaos) 是一个工具 **模拟容器中的时钟扭曲，测试它如何影响您的应用而不影响整个节点** 这样，我们就能够确切地查明时钟扭曲的潜在后果，并采取相应措施。

## 我们已经探索的模拟时钟扭曲的各种方法

在审查现有的选择时，我们清楚地知道，这些选择不能适用于在库伯内特运行的Chaos Mesh。 两种常见的方式模拟时钟偏差--直接改变节点时钟，并使用 Jepsen 框架--改变节点上所有过程的时间。 这些都不是我们可以接受的解决办法。 在 Kubernetes 容器中，如果我们注入一个影响整个节点的时钟偏差错误，同一节点上的其他容器将被打扰。 这种模棱两可的做法是不可容忍的。

那么，我们应该如何处理这个问题？ 好吧，我们心中的第一件事是使用 [伯克利包过滤器](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter) (BPF) 在内核中找到解决方案。

### `LD_PRELOAD`

`LD_PRELOAD` 是一个 Linux 环境变量，可以让您在程序执行之前定义哪些动态链接库。

此变量有两个优点：

- 我们可以调用我们自己的功能，而不知道源代码。
- 我们可以将代码注入其他程序以实现特定目的。

对于使用应用程序来调用glibc时间函数的某些语言 例如Rust 和 C，使用 `LD_PRELOAD` 足以模拟时钟扭曲。 但对戈兰高地来说，事情更加棘手。 因为诸如Golang等语言直接解析虚拟动态共享对象([vDSO](http://man7.org/linux/man-pages/man7/vdso.7.html))，一个加速系统调用的机制。 要获取时间函数地址，我们不能简单地使用 `LD_PRELOAD` 来拦截玻璃接口。 因此， `LD_PRELOAD` 不是我们的解决方案。

### 使用 BPF 修改 `时钟时间` 系统调用的返回值

我们还试图用BPF过滤任务 [进程识别号码](http://www.linfo.org/pid.html) (PID)。 这样，我们可以在指定过程中模拟时钟扭曲，并修改 `时钟时间` 系统调用的返回值。

这似乎是一个好主意，但我们也遇到了一个问题：在大多数情况下， vDSO 可以加快 `时钟时间`, 但 `时钟时间` 没有进行系统调用。 这个选择也不起作用。 糟糕。

所幸的是，我们确定如果系统内核版本是 4。 8 或更晚，如果我们使用 [HPET](https://www.kernel.org/doc/html/latest/timers/hpet.html) 时钟， `时钟_gettime()` 通过正常的系统调用而不是vDSO获得时间。 我们使用此方法实现了 [个时钟偏差](https://github.com/chaos-mesh/bpfki) 版本，它对Rust和C很有效。 至于Golang，程序可以获得正确的时间，但如果我们执行 `睡眠` 时钟倾斜注入， 睡眠操作很可能被屏蔽。 即使在注入被取消后，系统也无法恢复。 因此，我们也必须放弃这种做法。

## TimeChaos，我们的最终哈克

从上一节我们知道程序通常通过调用 `clock_gettime` 来获得系统时间。 在我们的情况下， `时钟时间` 使用 vDSO 来加速呼叫过程。 所以我们不能使用 `LD_PRELOAD` 来截取 `时钟时间` 系统调用。

我们找出了原因，然后解决了什么？ 从 vDSO 开始。 如果我们可以将存储为 `clock_gettime` vDSO 返回值的地址重定向到我们定义的地址， 我们可以解决这个问题。

说得比做得更容易。 为了实现这一目标，我们必须解决以下问题：

- 知道vDSO使用的用户模式地址
- 如果我们想要通过内核模式中的任何地址修改vDSO中的 `clock_gettime` 函数 vDSO的内核模式地址
- 了解如何修改 vDSO 数据

首先，我们需要在vDSO内部紧紧紧。 我们可以在 `/proc/pid/map` 中看到vDSO 内存地址。

```
$ cat /proc/pid/map
...
7ffe53143000-7ffe53145000 r-xp 00000000 00:00 0                     [vdso]
```

最后一行是 vDSO 信息。 此内存空间的权限是 `r-xp`: 可读和可执行，但不可写。 这意味着用户模式不能修改此内存。 我们可以使用 [ptrace](http://man7.org/linux/man-pages/man2/ptrace.2.html) 来避免这种限制。

接着，我们使用 `gdb 转储内存` 来导出vDSO，并使用 `objdump` 来查看内部情况。 下面是我们得到的东西：

```
(gdb) 转储内存vdso.so 0x007ffe531430000 0x007ffe53145000
$ objdump -T vdso.so vdso.so vddso.so vx0x007ffe531430000 0x0000007ffe535000
vdso. o: file format elf64-x86-64
DYNAMIC SYMBOL TABLE:
ffffffffffff700600 w DF .text 0000000000000545 LINUX_2.6时钟_gettime
```

我们可以看到整个vDSO就像一个 `。 o` 文件，所以我们可以使用可执行和可链接的格式 (ELF) 文件来格式化它。 有了这个信息，实现TimeChaos的基本工作流开始变形：

![时间Chaos 工作流](/img/timechaos-workflow.jpg)

<div class="caption-center"> 时间Chaos 工作流 </div>

上面的图表是 **TimeChaos**的过程，是Chaos Mesh的时钟扭曲的实现。

1. 使用 ptrace 来附加指定的 PID 进程来停止当前进程。
2. 使用 ptrace 在调用过程的虚拟地址空间中创建一个新映射，并使用 [`process_vm_writev`](https://linux.die.net/man/2/process_vm_writev) 写入我们定义为内存空间的 `fake_clock_gettime` 函数。
3. 使用 `process_vm_writev` 将指定的参数写入 `fake_clock_gettime`。 这些参数是我们想注入的时间，例如两小时后退或两天。
4. 使用 ptract 来修改 vDSO 中的 `时钟时间` 函数，并重定向到 `fake_clock_gettime` 函数。
5. 使用 ptract 断开PID 进程。

如果您对详细信息感兴趣，请参阅 [Chaos Mesh GitHub 版本库](https://github.com/chaos-mesh/chaos-mesh/blob/master/pkg/time/time_linux.go)。

## 在分布式SQL数据库模拟时钟偏差

统计数字说得很多。 我们将尝试在 [TiDB](https://pingcap.com/docs/stable/overview/)上的 TimeChaos ，一个开源， [NewSQL](https://en.wikipedia.org/wiki/NewSQL), 分布式的 SQL 数据库支持 [混合交易/分析处理](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing) (HTTP) 工作量，以查看混乱测试是否真正有效。

TiDB 使用中央服务时间戳Oracle (TSO) 获取全球一致的版本号，并确保交易版本号单数增加。 停战监督组织的服务由安置驱动程序(PD)部分管理。 因此，我们选择一个随机的 PD 节点，并定期注入时间Chaos，每个节点有10毫秒后的时钟斜杠。 让我们看看TiDB是否能够迎接挑战。

为了更好地进行测试，我们使用 [银行](https://github.com/cwen0/bank) 作为工作负荷，模拟银行系统中的资金转移。 它常用来验证数据库交易的正确性。

这是我们的测试配置：

```
apiVersion: chaos-mesh。 rg/v1alpha1
种: TimeChaos
metadata:
  name: tidskew-example
  namespace: tidb-demo
spec:
  模式: one
  selector:
    labeleselector:
      "app. 伯尔尼。 o/component: "pd"
  timeOffset:
    sec: -600
  clockIds:
    - CLOCK_REALTIME
  duration: "10s"
  scheduler:
    cron: "@ever 1m"
```

在这次测试中，Chaos Mesh 将TimeChaos注入选定的 PD Pod 每1毫秒10秒。 在这段时间内，国防部获得的时间将从实际时间起抵销600秒。 更多详细信息，见 [Chaos Mesh Wiki](https://github.com/chaos-mesh/chaos-mesh/wiki/Time-Chaos)。

让我们使用 `kubectl 应用` 命令，创建 TimeChaos 实验：

```
kubectl 应用 -f pd-time.yaml
```

现在，我们可以通过以下命令检索PD日志：

```
kubectl log-n tidb-demo tidb-app-pd-0 | grep "system time jump backward"
```

下面是日志：

```
[2020/03/24 09:06:23.164 +00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=158504138306010693]
[20/03/24 09:16:32.260 +00] [ERROR] [systime_mon. o:32] ["system time jump backward"] [last=15850419921604762]
[2020/03/24 09:20:32.059 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042231960027622]
[20/03/24 09:23:32] 59 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042411960079655]
[2020/03/24 09:25:32. 59 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042531963640321]
[20/03/24 09:28:32. 60 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042711960148191]
[20/03/24 09:33:32.063 +00] [ERROR] [systime_mon. o:32] ["system time jump backward"] [last=1585043011960517655]
[20/03/24 09:34:32.060 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043071959942937]
[20/03/24 09:35:32] 59 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043131978582964]
[20/03/24 09:36:32.059 +00] [ERROR] [stime_mon. o:32] ["system time jump backward"] [last=1585043191960687755]
[20/03/24 09:38:32.060 +00:00] [ERROR] [stime_mon.go:32] ["system time jump backward"] [last=158504311959970737]
[20/03/24 09:41:32] 60 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043491959970502]
[2020/03/24 09:45:32] 61 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043731961304629]
...
```

从上面的日志，我们看到现在和之后每隔一段时间，PD 都会检测到系统的时间倒退。 这意味着：

- TimeChaos成功模拟时钟扭曲。
- 和平民主党可以处理时钟偏差的情况。

这是令人鼓舞的。 但TimeChaos是否影响PD以外的服务？ 我们可以在Chaos仪表板中查看它：

![Chaos 仪表板](/img/chaos-dashboard.jpg)

<div class="caption-center"> Chaos 仪表板 </div>

很清楚，在监视器中，TimeChaos注入每1毫秒，整个持续时间持续10秒。 此外，TiDB没有受到这种注射的影响。 银行方案正常运作，业绩没有受到影响。

## 试试Chaos Mesh

Chaos Mesh 作为一个云端本地的chaos工程平台，其特点是

Kubernetes 上复杂系统的 [ 个故障注入方法](https://pingcap.com/blog/chaos-mesh-your-chaos-engineering-solution-for-system-resiliency-on-kubernetes/)， 覆盖Pods、 网络、文件系统，甚至内核中的错误。</p> 

想要在混乱工程方面有一些实践经验吗？ 欢迎使用 [Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)。 此 [十分钟的教程](https://pingcap.com/blog/run-first-chaos-experiment-in-ten-minutes/) 将帮助您快速启动混乱工程并运行您第一次与Chaos Mesh的chaos试验。
