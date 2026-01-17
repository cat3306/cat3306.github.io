# rust 跨平台编译


<!--more-->
### 支持的平台
| 目标平台                | target triple                |
| ------------------- | ---------------------------- |
| Linux x86_64        | `x86_64-unknown-linux-gnu`   |
| Linux x86_64 (静态)   | `x86_64-unknown-linux-musl`  |
| Linux ARM64         | `aarch64-unknown-linux-gnu`  |
| Linux ARM64 (静态)    | `aarch64-unknown-linux-musl` |
| macOS Intel         | `x86_64-apple-darwin`        |
| macOS Apple Silicon | `aarch64-apple-darwin`       |
| Windows x64         | `x86_64-pc-windows-gnu`      |
| Windows MSVC        | `x86_64-pc-windows-msvc`     |

## 在 Mac ARM 架构上跨平台编译 Rust 到 Linux，不依赖 Docker 和 glibc，你需要使用 musl 作为 C 标准库。以下是详细步骤：
``` bash
1. 安装交叉编译工具链

bash# 安装 Rust 工具链

rustup target add x86_64-unknown-linux-musl
rustup target add aarch64-unknown-linux-musl


# 安装 musl-cross 工具链（通过 Homebrew）

brew install filosottile/musl-cross/musl-cross


2. 配置 Cargo
创建或编辑 ~/.cargo/config.toml：
toml[target.x86_64-unknown-linux-musl]
linker = "x86_64-linux-musl-gcc"

[target.aarch64-unknown-linux-musl]
linker = "aarch64-linux-musl-gcc"


3. 编译项目
bash# 编译 x86_64 Linux 版本

cargo build --release --target x86_64-unknown-linux-musl

# 编译 ARM64 Linux 版本
cargo build --release --target aarch64-unknown-linux-musl


4. 处理依赖问题
如果你的项目依赖了需要 C 库的 crate（如 OpenSSL），有两种方案：
方案 A：使用纯 Rust 替代品

用 rustls 替代 openssl
用 ring 替代加密库

方案 B：静态链接 C 库
bash# 安装额外的构建工具
brew install messense/macos-cross-toolchains/x86_64-unknown-linux-musl

# 设置环境变量
export CC_x86_64_unknown_linux_musl=x86_64-linux-musl-gcc
export CXX_x86_64_unknown_linux_musl=x86_64-linux-musl-g++
export AR_x86_64_unknown_linux_musl=x86_64-linux-musl-ar
export CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER=x86_64-linux-musl-gcc
5. 验证编译结果
bash# 检查编译的二进制文件
file target/x86_64-unknown-linux-musl/release/your_binary

# 应该显示类似：
# ELF 64-bit LSB executable, x86-64, statically linked
```

