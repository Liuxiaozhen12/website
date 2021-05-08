---
id: 内核chaos_expert
title: KernelChaos 测试
sidebar_label: KernelChaos 测试
---

本文档描述了如何在 Chaos Mesh中创建 KernelChaos 实验。

虽然KernelChaos的目标是某个药水，但其他药水的性能也受到影响，这取决于具体的呼叫链和频率。 这是因为同一个主机的所有坑都共享同一个内核。

> **警告：**
> 
> 此功能默认被禁用。 不在生产环境中使用它。

## 必备条件

- Linux 内核：版本 >= 4.18
- [CONFIG_BP_KPROBE_OVERRIDE](https://cateee.net/lkddb/web-lkddb/BPF_KPROBE_OVERRIDE.html) 已启用
- `bpfki.create = true` in [values.yaml](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/values.yaml)

## 配置文件

下面是 KernelChaos 配置文件样本：

```yaml
apiVersion: chaos-mesh。 rg/v1alpha1
kind: KernelChaos
metadata:
  name: kernel-chaos-example
  namespace: chaos-testing
spec:
  mode: one
  selector:
    namespace:
      - chaos-mount
  failKernRequest:
    callchain:
      - funcname: '__x64_sys_mount'
    failtype: 0
```

欲了解更多样本文件，请参阅 [示例](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)。 您可以根据需要编辑它们。

描述：

- **模式** 定义了选择pods的模式。
- **选择器** 指定了用于故障注入的目标点。 欲了解更多详情，请参阅 [定义Chaos 实验范围](experiment_scope.md)。
- **故障内核请求** 定义了指定的注入模式 (kmalloc、bio等)，有一个通话链和一套可选的预测集。 字段为：

  - **故障输入** 表示要失败的内容，可以设置为 `0` / `1` / `2`。

    - 如果 `0`, 指示板块失败 (should_failslab)
    - 如果 `1`, 指示分配页面失败 (should_fail_alloc_page)
    - 如果 `2`, 指示失败的 bio (should_fail_bio)

    欲了解更多信息，请参阅 [故障注入](https://www.kernel.org/doc/html/latest/fault-injection/fault-injection.html) and [注入示例](http://github.com/iovisor/bcc/blob/master/tools/inject_example.txt)。

  - **调用链** 表示一个特殊的调用链，例如：

    ```c
    ext4_mount
    -> mount_subtree
       ->...
          -> should_failslab
    ```

    有一套可选的预测和一套可选的参数，它们与预测一起使用。 查看 [通话链和前提示例](https://github.com/chaos-mesh/bpfki/tree/develop/examples) 了解更多信息。 如果没有特殊通话链，请保持 `通话链` 为空。 这意味着它在任何通话链中都会使用sabb alloc合金失败(例如，kmalloc)。

    Schallchain的类型是一个框架数组，框架有三个字段：

    - **函数名称** 可以从内核源找到 `/proc/kallsyms`, 例如 `ext4_mount`。
    - **参数** 被用于预测，例如，如果您想在 `d_alloc_parabel(struct dentry *parent, const struct qstr *name)` with a special name `bananas`, 您需要将其设置为 `结构密度*父母，const struct qstr *name`否则省略它。
    - **前提** 访问此帧的参数，例如参数，您可以将其设置为 `STRNCMP(name->名称) "香蕉", 8"` 只注入它, 或省略它以注入所有 d_alloc_partial 通话链。

  - **头** 表示您需要的内核头部。 Eg: "linux/mmzone.h", "linux/blkdev.h" 等.
  - **概率** 表示失败的概率。 如果您想要1%，请设置此字段为 `1`。
  - **次** 表示失败的最大时间。

- **持续时间** 定义了每次混乱状态实验的持续时间。 在上面的样本文件中，时间混乱持续10秒。
- **调度器** 定义了混乱状态实验运行时间的调度规则。 欲了解更多规则信息，请参阅 [robfig/cron](https://godoc.org/github.com/robfig/cron)

## 用法

KernelChaos的函数与 [注入.py](https://github.com/iovisor/bcc/blob/master/tools/inject.py)相似，它保证了特定注入模式(kmalloc、bio等)的适当返回错误。 给出一个调用链和可选的预测集。

您可以读取 [注入示例.txt](https://github.com/iovisor/bcc/blob/master/tools/inject_example.txt) 了解更多信息。

下面是一个样本程序：

```c
#include <sys/mount.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

int main(void) v.
    int ret;
    当(1) v.
        ret = mount("/dev/sdc", "/mnt", "ext4",
                MS_MGC_VAL | MS_RDONLY | MS_NOSUID, "");
        如果(ret < 0)
            fprintf(stderr, "%s\n", streror(errno));
        睡眠(1)；
        ret = umount("/mnt")；
        if (ret < 0)
            fprintf(stderr, "%s\n", streror(errno));
    }
}
```

在注入过程中，输出与此相似：

```
> 无法分配内存
> 无效的参数
> 无法分配内存
> 无效的参数
> 无法分配内存
> 无效的参数
> 无法分配内存
> 无效的参数
> 无法分配内存
> 无效的参数
```

## 限制

虽然我们使用容器ID来限制故障注入，但是某些行为可能会触发系统行为。 例如：

当 `故障类型` 是 `1`, 这意味着实际页面分配将失败。 如果行为在很短时间内是连续的(例如：``while (1) {memset(malloc(1M), '1', 1M)}`)，系统oom-killer将被唤醒以释放内存。 所以容器ID将会失去雾杀手的极限。
