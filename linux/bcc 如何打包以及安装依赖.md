---
tags:
  - ebpf
  - bcc
---



未验证, ai 生成

---
在 Ubuntu 上安装 **BCC（BPF Compiler Collection）** 并运行基于 BPF 的程序（无论是 C++ 还是 Rust）需要一些依赖和步骤。以下是详细的安装和打包指南。

---

## 1. **安装 BCC 及其依赖**
BCC 是一个用于 Linux 内核 BPF（Berkeley Packet Filter）编程的工具集，允许用户编写高效的内核跟踪和监控程序。

### （1）安装依赖
在 Ubuntu 上安装 BCC 需要以下依赖：
- 内核头文件
- Clang/LLVM（用于编译 BPF 程序）
- Python（用于运行 BCC 工具）

运行以下命令安装依赖：
```bash
sudo apt update
sudo apt install -y bpfcc-tools libbpfcc-dev linux-headers-$(uname -r) clang llvm libelf-dev libfl-dev zlib1g-dev
```

### （2）安装 BCC
从 Ubuntu 20.04 开始，BCC 可以直接通过包管理器安装：
```bash
sudo apt install -y bpfcc-tools
```

如果需要最新版本的 BCC，可以从源码编译安装：
```bash
# 安装编译依赖
sudo apt install -y git cmake python3-distutils

# 克隆 BCC 源码
git clone https://github.com/iovisor/bcc.git
cd bcc

# 编译和安装
mkdir build
cd build
cmake ..
make
sudo make install
```

### （3）验证安装
安装完成后，可以通过以下命令验证 BCC 是否安装成功：
```bash
sudo /usr/share/bcc/tools/execsnoop
```
如果看到实时输出的进程执行信息，说明 BCC 安装成功。

---

## 2. **C++ 程序如何打包**
如果使用 C++ 编写 BPF 程序并通过 BCC 运行，可以按照以下步骤打包：

### （1）编写 C++ 程序
假设你有一个 C++ 程序 `main.cpp`，它使用 BCC 库加载和运行 BPF 程序：
```cpp
#include <iostream>
#include "bcc/BPF.h"

int main() {
    ebpf::BPF bpf;
    auto res = bpf.init("BPF_PROGRAM");
    if (res.code() != 0) {
        std::cerr << "Failed to load BPF program: " << res.msg() << std::endl;
        return 1;
    }
    std::cout << "BPF program loaded successfully!" << std::endl;
    return 0;
}
```

### （2）编译 C++ 程序
使用以下命令编译 C++ 程序：
```bash
g++ -std=c++11 -o my_bpf_program main.cpp -lbcc
```

### （3）打包程序
将编译后的可执行文件和相关依赖打包：
```bash
# 创建打包目录
mkdir -p my_bpf_program_pkg
cp my_bpf_program my_bpf_program_pkg/

# 打包为 tar.gz
tar -czvf my_bpf_program_pkg.tar.gz my_bpf_program_pkg
```

---

## 3. **Rust 程序如何打包**
如果使用 Rust 编写 BPF 程序并通过 BCC 运行，可以按照以下步骤打包：

### （1）添加 BCC 依赖
在 `Cargo.toml` 中添加 BCC 的 Rust 绑定库：
```toml
[dependencies]
bcc = "0.10"
```

### （2）编写 Rust 程序
假设你有一个 Rust 程序 `main.rs`，它使用 BCC 库加载和运行 BPF 程序：
```rust
use bcc::BccError;
use bcc::core::BPF;

fn main() -> Result<(), BccError> {
    let code = "BPF_PROGRAM";
    let bpf = BPF::new(code)?;
    println!("BPF program loaded successfully!");
    Ok(())
}
```

### （3）编译 Rust 程序
使用以下命令编译 Rust 程序：
```bash
cargo build --release
```

### （4）打包程序
将编译后的可执行文件和相关依赖打包：
```bash
# 创建打包目录
mkdir -p my_bpf_program_pkg
cp target/release/my_bpf_program my_bpf_program_pkg/

# 打包为 tar.gz
tar -czvf my_bpf_program_pkg.tar.gz my_bpf_program_pkg
```

---

## 4. **运行打包的程序**
将打包的程序分发到目标机器后，解压并运行：
```bash
# 解压
tar -xzvf my_bpf_program_pkg.tar.gz

# 运行
cd my_bpf_program_pkg
./my_bpf_program
```

---

## 5. **注意事项**
- **内核版本**：BCC 需要较新的 Linux 内核（4.1+），建议使用 Ubuntu 20.04 或更高版本。
- **权限**：运行 BPF 程序需要 root 权限。
- **调试工具**：可以使用 `bpftool` 或 `bcc` 自带的工具（如 `trace`、`argdist`）调试 BPF 程序。

---

## 6. **总结**
在 Ubuntu 上安装 BCC 并运行基于 BPF 的程序（无论是 C++ 还是 Rust）需要安装相关依赖，并通过编译和打包工具生成可执行文件。通过合理管理依赖和打包流程，可以轻松分发和部署 BPF 程序。如果需要更详细的示例或进一步的学习资源，可以参考 BCC 官方文档和示例代码。
