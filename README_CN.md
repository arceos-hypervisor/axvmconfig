<h1 align="center">axvmconfig</h1>

<p align="center">虚拟机配置工具与库</p>

<div align="center">

[![Crates.io](https://img.shields.io/crates/v/axvmconfig.svg)](https://crates.io/crates/axvmconfig)
[![Docs.rs](https://docs.rs/axvmconfig/badge.svg)](https://docs.rs/axvmconfig)
[![Rust](https://img.shields.io/badge/edition-2021-orange.svg)](https://www.rust-lang.org/)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](https://github.com/arceos-hypervisor/axvmconfig/blob/main/LICENSE)

</div>

[English](README.md) | 中文

# Introduction

`axvmconfig` 是面向 AxVisor 的虚拟机配置工具与库。它提供 Rust 数据结构和命令行工具，用于在多种体系结构下解析、校验和生成基于 TOML 的虚拟机配置文件。

该库导出多个核心配置类型，包括：

- **`AxVMCrateConfig`** - 从 TOML 解析得到的顶层虚拟机配置
- **`VMType`** - 表示 HostVM、RTOS、Linux 等 guest 类型
- **`VmMemMappingType`** - 定义 guest 内存映射方式
- **`EmulatedDeviceType`** - 枚举受支持的模拟设备类型
- **`VMInterruptMode`** - 定义 `no_irq`、`emulated`、`passthrough` 等中断处理模式

当启用 `std` feature 时，该 crate 还提供 `axvmconfig` CLI 工具，用于检查已有配置和生成模板。

## Quick Start

### Requirements

- Rust nightly 工具链
- Rust 组件：rust-src、clippy、rustfmt

```bash
# 安装 rustup（如果尚未安装）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装 nightly 工具链与所需组件
rustup install nightly
rustup component add rust-src clippy rustfmt --toolchain nightly
```

### Run Check and Test

```bash
# 1. 进入仓库目录
cd axvmconfig

# 2. 代码检查
./scripts/check.sh

# 3. 运行测试
./scripts/test.sh
```

## Integration

### Installation

将以下依赖加入 `Cargo.toml`：

```toml
[dependencies]
axvmconfig = "0.2.2"
```

### Example

```rust
use axvmconfig::{
    AxVMCrateConfig, EmulatedDeviceType, VMInterruptMode, VMType, VmMemMappingType,
};

fn main() {
    let toml_str = r#"
[base]
id = 1
name = "guest"
vm_type = 1
cpu_num = 1

[kernel]
entry_point = 0x80200000
kernel_path = "guest.bin"
kernel_load_addr = 0x80200000
image_location = "fs"

memory_regions = [
    [0x80000000, 0x1000000, 0x7, 1],
]

[devices]
passthrough_devices = []
emu_devices = [
    ["console0", 0x10000000, 0x1000, 0, 0x2, []],
]
interrupt_mode = "emulated"
"#;

    let config = AxVMCrateConfig::from_toml(toml_str).unwrap();

    assert_eq!(config.base.vm_type, usize::from(VMType::VMTRTOS));
    assert_eq!(
        config.kernel.memory_regions[0].map_type,
        VmMemMappingType::MapIdentical
    );
    assert_eq!(config.devices.emu_devices[0].emu_type, EmulatedDeviceType::Console);
    assert_eq!(config.devices.interrupt_mode, VMInterruptMode::Emulated);
}
```

### Documentation

生成并查看 API 文档：

```bash
cargo doc --no-deps --open
```

在线文档： [docs.rs/axvmconfig](https://docs.rs/axvmconfig)

# Contributing

1. Fork 仓库并创建分支
2. 本地运行检查：`./scripts/check.sh`
3. 本地运行测试：`./scripts/test.sh`
4. 提交 PR 并通过 CI 检查

# License

本项目基于 Apache License 2.0 许可证发布。详见 [LICENSE](LICENSE)。
