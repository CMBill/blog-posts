---
title: cargo install 的一些杂项
published: 2026-05-17
description: cargo install 的常用命令，以及 cargo-binstall 和 cargo-update 等辅助工具的用法。
tags: [Rust, Cargo, 工具链, AIGC]
category: 开发工具
slug: cargo-install-notes
draft: false
---

​`cargo install`​ 用于从 [crates.io](https://crates.io/) 或指定的 Git 仓库下载 Rust crate 的源代码，并在本地编译后将其二进制文件安装到 `~/.cargo/bin` 目录中。

- ​**安装一个工具**：  

  ```bash
  cargo install ripgrep
  ```
- ​**安装特定版本**：  

  ```
  cargo install ripgrep@13.0.0
  ```
- ​**从 Git 仓库安装**：  

  ```bash
  cargo install --git https://github.com/BurntSushi/ripgrep
  ```
- ​**列出已安装的包**：  

  ```bash
  cargo install --list
  ```
- ​**卸载一个工具**：  

  ```
  cargo uninstall ripgrep
  ```

> 💡 ​**注意**​：`cargo install`​ 默认会将二进制文件输出到 `~/.cargo/bin`​，请确保该目录已添加到系统的 `PATH` 环境变量中。

## ​`--locked` 参数

​`cargo install`​ 在默认情况下**不会**使用 crate 源码中可能自带的 `Cargo.lock`​ 文件，而是根据 `Cargo.toml`​ 中的依赖范围重新计算并​**取满足条件的最新依赖版本**。这样做在某些情况下可能引入不兼容的改动，导致编译失败或运行时异常。

添加 `--locked` 参数可以改变这一行为：

```bash
cargo install --locked ripgrep
```

其效果如下：

- **强制要求**源码目录中存在 `Cargo.lock` 文件（若缺失，Cargo 会报错退出）。
- **严格遵守** `Cargo.lock` 中记录的每个依赖的具体版本，不再重新解析。

这带来两点好处：

1. **可重现的安装** – 安装结果与 crate 作者在开发及发布时所用的依赖环境完全一致。
2. **更高的稳定性** – 避免因某个依赖的小幅更新（即使语义上兼容）意外破坏工具功能。

建议在安装绝大多数成熟的命令行工具（如 `ripgrep`​、`fd`​、`bat`​）时都加上 `--locked`。

## 使用`cargo-binstall`直接安装二进制文件

​`cargo-binstall`​ 是一个旨在**替代** `cargo install`​ 的第三方工具。它通过**下载预编译的二进制文件**来代替本地编译，从而极大缩短安装时间。

### ✨ 为什么使用它？

| 特性     | ​`cargo binstall`              | ​`cargo install`           |
| -------- | ------------------------------ | -------------------------- |
| 安装方式 | 下载预编译二进制               | 下载源码并本地编译         |
| 速度     | 几秒内完成（尤其适合大型项目） | 耗时较长，需要 Rust 工具链 |
| 依赖要求 | 不需要本地编译工具链           | 必须安装 Rust 工具链       |

### 🚀 如何使用

首先，通过 `cargo install`​ 安装 `cargo-binstall` 自身（这一步仍需编译，但只需一次）：

```bash
cargo install cargo-binstall
```

之后，即可使用 `cargo binstall` 安装其他工具：

```bash
cargo binstall ripgrep        # 快速安装，自动获取预编译版本
cargo binstall tokei@12.1.0   # 指定版本
```

如果 `cargo-binstall`​ 找不到预编译的二进制文件，它会**自动回退**到源码编译（即执行 `cargo install`），从而保证安装成功率。

### ⚙️ 工作原理

​`cargo binstall` 按以下顺序尝试获取二进制：

1. **GitHub Releases** – 检查 crate 的 GitHub Release 页面是否附带了与当前平台匹配的预编译二进制。
2. **cargo-quickinstall** – 查询 `cargo-quickinstall` 提供的预编译包仓库。
3. **回退到源码安装** – 如果以上都失败，则执行 `cargo install`。

​`cargo-binstall` 为 CI 环境、资源受限设备或追求效率的开发者提供了非常友好的体验。

## 使用`cargo-update`更新

​`cargo install`​ 本身**不提供**内置的更新命令。当你需要将已安装的工具升级到最新版本时，可以使用第三方工具 `cargo-update`。

### 📦 安装 `cargo-update`

bash

```
cargo install cargo-update
```

### 🔄 更新所有已安装的包

bash

```
cargo install-update -a
```

​`-a`​ 表示“全部”。该命令会检查每个通过 `cargo install` 安装的包，若发现新版本则自动升级。

### 🎯 更新特定包

bash

```
cargo install-update ripgrep fd-find
```

### 💡 注意事项

- ​`cargo-update`​ 会跳过那些不是通过 `cargo install` 安装的程序（例如系统包管理器安装的）。
- ​`cargo-update`​ 自身也可以自我更新：再次运行 `cargo install cargo-update`​ 或执行 `cargo install-update cargo-update`。

### 🆚 与 `cargo binstall` 的协作

- ​**安装/重装**​：优先使用 `cargo binstall` 达到快速部署。
- ​**更新**​：仍然使用 `cargo install-update`​，因为它会智能地处理版本检查，无论当初是通过 `cargo install`​ 还是 `cargo binstall` 安装的，都能正确升级（前提是后续更新时依然能获取到二进制或源码）。
