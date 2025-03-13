---
tags:
  - kprobe
  - ebpf
---
**动态插桩（Dynamic Instrumentation）** 是一种在运行时动态修改程序行为的调试和分析技术。在 Linux 系统中，**kprobe** 和 **uprobe** 是两种常用的动态插桩工具，分别用于内核空间和用户空间的插桩。以下是它们的详细介绍和使用方法。

---

## 1. **kprobe（内核探针）**
kprobe 是一种内核动态插桩技术，允许在内核函数的入口和出口处插入探针，从而监控和调试内核行为。

### （1）**kprobe 的工作原理**
- **入口探针（kprobe）**：在内核函数的入口处插入探针，记录函数调用时的上下文（如寄存器、参数）。
- **出口探针（kretprobe）**：在内核函数的出口处插入探针，记录函数的返回值。

### （2）**kprobe 的使用场景**
- **内核调试**：监控内核函数的调用和返回值。
- **性能分析**：统计内核函数的执行时间和调用频率。
- **安全监控**：检测内核中的异常行为。

### （3）**kprobe 的使用示例**
以下是一个使用 kprobe 监控 `do_sys_open` 内核函数的示例：

#### 使用 BCC 工具
BCC 提供了简单易用的 Python 接口来使用 kprobe：
```python
from bcc import BPF

# 定义 BPF 程序
bpf_text = """
#include <uapi/linux/ptrace.h>

int kprobe__do_sys_open(struct pt_regs *ctx, const char __user *filename, int flags, umode_t mode) {
    bpf_trace_printk("do_sys_open: %s\\n", filename);
    return 0;
}
"""

# 加载 BPF 程序
b = BPF(text=bpf_text)

# 输出跟踪信息
print("Tracing do_sys_open... Ctrl+C to exit.")
b.trace_print()
```

#### 使用内核模块
在内核模块中使用 kprobe：
```c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>

static struct kprobe kp = {
    .symbol_name = "do_sys_open",
};

static int handler_pre(struct kprobe *p, struct pt_regs *regs) {
    pr_info("do_sys_open called\\n");
    return 0;
}

static void handler_post(struct kprobe *p, struct pt_regs *regs, unsigned long flags) {
    pr_info("do_sys_open returned\\n");
}

static int __init kprobe_init(void) {
    kp.pre_handler = handler_pre;
    kp.post_handler = handler_post;
    register_kprobe(&kp);
    return 0;
}

static void __exit kprobe_exit(void) {
    unregister_kprobe(&kp);
}

module_init(kprobe_init)
module_exit(kprobe_exit)
MODULE_LICENSE("GPL");
```

---

## 2. **uprobe（用户空间探针）**
uprobe 是一种用户空间动态插桩技术，允许在用户空间程序的函数入口和出口处插入探针，从而监控和调试用户空间程序。

### （1）**uprobe 的工作原理**
- **入口探针（uprobe）**：在用户空间函数的入口处插入探针，记录函数调用时的上下文（如寄存器、参数）。
- **出口探针（uretprobe）**：在用户空间函数的出口处插入探针，记录函数的返回值。

### （2）**uprobe 的使用场景**
- **用户空间调试**：监控用户空间函数的调用和返回值。
- **性能分析**：统计用户空间函数的执行时间和调用频率。
- **动态追踪**：实时监控用户空间程序的行为。

### （3）**uprobe 的使用示例**
以下是一个使用 uprobe 监控用户空间函数 `malloc` 的示例：

#### 使用 BCC 工具
BCC 提供了简单易用的 Python 接口来使用 uprobe：
```python
from bcc import BPF

# 定义 BPF 程序
bpf_text = """
#include <uapi/linux/ptrace.h>

int uprobe__malloc(struct pt_regs *ctx, size_t size) {
    bpf_trace_printk("malloc called: size=%lu\\n", size);
    return 0;
}
"""

# 加载 BPF 程序
b = BPF(text=bpf_text)

# 附加 uprobe
b.attach_uprobe(name="c", sym="malloc", fn_name="uprobe__malloc")

# 输出跟踪信息
print("Tracing malloc... Ctrl+C to exit.")
b.trace_print()
```

#### 使用内核模块
在内核模块中使用 uprobe：
```c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/uprobes.h>

static struct uprobe up = {
    .symbol_name = "malloc",
};

static int handler_pre(struct uprobe *up, struct pt_regs *regs) {
    pr_info("malloc called\\n");
    return 0;
}

static void handler_post(struct uprobe *up, struct pt_regs *regs, unsigned long flags) {
    pr_info("malloc returned\\n");
}

static int __init uprobe_init(void) {
    up.pre_handler = handler_pre;
    up.post_handler = handler_post;
    register_uprobe(&up);
    return 0;
}

static void __exit uprobe_exit(void) {
    unregister_uprobe(&up);
}

module_init(uprobe_init)
module_exit(uprobe_exit)
MODULE_LICENSE("GPL");
```

---

## 3. **kprobe 和 uprobe 的对比**
| 特性                | kprobe                          | uprobe                          |
|---------------------|---------------------------------|---------------------------------|
| **插桩对象**         | 内核函数                        | 用户空间函数                    |
| **使用场景**         | 内核调试、性能分析、安全监控     | 用户空间调试、性能分析、动态追踪 |
| **实现方式**         | 内核模块或 BCC 工具             | 内核模块或 BCC 工具             |
| **性能影响**         | 对内核性能有一定影响            | 对用户空间程序性能有一定影响    |
| **复杂性**           | 较高                            | 较低                            |

---

## 4. **总结**
- **kprobe** 用于内核空间的动态插桩，适合调试和分析内核行为。
- **uprobe** 用于用户空间的动态插桩，适合调试和分析用户空间程序。
- 通过 BCC 工具或内核模块，可以方便地使用 kprobe 和 uprobe 进行动态插桩。

如果需要更详细的示例或进一步的学习资源，可以参考以下文档：
- [BCC 官方文档](https://github.com/iovisor/bcc)
- [Linux 内核文档 - kprobe](https://www.kernel.org/doc/html/latest/trace/kprobes.html)
- [Linux 内核文档 - uprobe](https://www.kernel.org/doc/html/latest/trace/uprobetracer.html)
