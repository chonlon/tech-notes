---
tags:
  - kprobe
  - uprobe
  - 符号
---
是的，无论是 **kprobe** 还是 **uprobe**，通常都需要符号信息（Symbol Information）来正确插桩。符号信息是程序或内核中函数和变量的名称与地址的映射关系，动态插桩工具依赖这些信息来定位需要插桩的函数或代码位置。

以下是关于符号信息的详细说明以及如何确保正确插桩的注意事项：

---

## 1. **为什么需要符号信息？**
动态插桩工具（如 kprobe 和 uprobe）需要知道目标函数或代码的位置（地址）才能插入探针。符号信息提供了以下关键信息：
- **函数名称**：如 `do_sys_open`（内核函数）或 `malloc`（用户空间函数）。
- **函数地址**：函数在内存中的实际地址。
- **调试信息**：如参数类型、返回值类型等（可选）。

如果没有符号信息，插桩工具无法确定目标函数的位置，从而导致插桩失败。

---

## 2. **kprobe 的符号需求**
kprobe 用于内核空间的插桩，通常需要内核符号表（Kernel Symbol Table）来定位内核函数。

### （1）内核符号表
- 内核符号表包含了内核中所有导出函数和变量的名称与地址。
- 可以通过 `/proc/kallsyms` 文件查看内核符号表：
  ```bash
  cat /proc/kallsyms | grep do_sys_open
  ```
  输出示例：
  ```
  ffffffff81234560 T do_sys_open
  ```

### （2）确保内核符号可用
- **启用内核调试信息**：在编译内核时，确保启用了 `CONFIG_KALLSYMS` 和 `CONFIG_KALLSYMS_ALL` 选项。
- **加载内核符号**：如果使用内核模块，确保模块的符号信息已加载：
  ```bash
  sudo insmod my_module.ko
  sudo cat /proc/kallsyms | grep my_function
  ```

### （3）kprobe 示例
以下是一个使用 kprobe 插桩 `do_sys_open` 的示例：
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

---

## 3. **uprobe 的符号需求**
uprobe 用于用户空间的插桩，通常需要目标程序的符号表（Symbol Table）来定位用户空间函数。

### （1）用户空间符号表
- 用户空间程序的符号表通常包含在可执行文件或共享库中。
- 可以通过 `nm` 或 `objdump` 查看符号表：
  ```bash
  nm /lib/x86_64-linux-gnu/libc.so.6 | grep malloc
  ```
  输出示例：
  ```
  0000000000095c40 T malloc
  ```

### （2）确保用户空间符号可用
- **编译时保留符号**：在编译用户空间程序时，确保未剥离符号表（不要使用 `strip` 命令）。
- **调试信息**：如果需要参数类型等调试信息，可以编译时添加 `-g` 选项：
  ```bash
  gcc -g -o my_program my_program.c
  ```

### （3）uprobe 示例
以下是一个使用 uprobe 插桩 `malloc` 的示例：
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

---

## 4. **没有符号信息时的替代方案**
如果目标程序或内核没有符号信息，仍然可以通过以下方式插桩：
- **地址插桩**：直接指定目标函数的地址（而不是名称）。
  - 对于 kprobe，可以通过 `/proc/kallsyms` 查找函数地址。
  - 对于 uprobe，可以通过反汇编工具（如 `objdump`）查找函数地址。
- **动态符号解析**：使用工具（如 `dlopen` 和 `dlsym`）在运行时解析符号地址。

### 示例：地址插桩
假设 `do_sys_open` 的地址是 `0xffffffff81234560`，可以使用以下方式插桩：
```python
from bcc import BPF

# 定义 BPF 程序
bpf_text = """
#include <uapi/linux/ptrace.h>

int kprobe__do_sys_open(struct pt_regs *ctx) {
    bpf_trace_printk("do_sys_open called\\n");
    return 0;
}
"""

# 加载 BPF 程序
b = BPF(text=bpf_text)

# 附加 kprobe（使用地址）
b.attach_kprobe(event="0xffffffff81234560", fn_name="kprobe__do_sys_open")

# 输出跟踪信息
print("Tracing do_sys_open... Ctrl+C to exit.")
b.trace_print()
```

---

## 5. **总结**
- **kprobe** 和 **uprobe** 都需要符号信息来正确插桩。
- 对于内核空间，确保内核符号表可用（通过 `/proc/kallsyms`）。
- 对于用户空间，确保目标程序的符号表未剥离（通过 `nm` 或 `objdump`）。
- 如果没有符号信息，可以通过地址插桩或动态符号解析实现插桩。

通过合理管理符号信息，可以确保动态插桩工具能够正确工作，从而实现对内核和用户空间程序的高效调试和分析。