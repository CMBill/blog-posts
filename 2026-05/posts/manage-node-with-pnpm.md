---
title: 使用 pnpm 管理 node 运行环境
published: 2026-05-17
description: 使用 pnpm 的 runtime 命令管理 Node.js 运行环境，包括安装不同版本和项目级配置说明。
tags: [pnpm, Node.js, 包管理, AIGC]
category: 开发工具
slug: manage-node-with-pnpm
draft: false
---

参考[安装 | pnpm](https://pnpm.io/zh/installation)，[pnpm runtime <cmd> | pnpm](https://pnpm.io/zh/cli/runtime)

在 pnpm v6 开始可以使用`pnpm env`​命令管理 Node 环境，类似于 nvm 等工具，而在 v11 开始变为`pnpm runtime`命令。因此，可以使用独立脚本在不安装 Node 的情况下直接安装 pnpm，然后来管理计算机上的 Node 环境。

## 基本用法

使用 `pnpm runtime set` 命令可以方便地安装各种版本：

```bash
pnpm runtime set <name> <version> [-g]
```

支持的运行时包括：

- ​`node` - Node.js
- ​`deno` - Deno
- ​`bun` - Bun

例如：

```bash
# 安装最新的 Node.js v22 长期支持（LTS）版本
pnpm runtime set node lts -g

# 安装最新版本
pnpm runtime set node latest -g

# 安装指定版本号
pnpm runtime set node 22.11.0 -g

# 安装预发布版 (Release Candidate)
pnpm runtime set node rc -g

# 通过代号安装，例如 Node.js 4.x 的代号 'argon'
pnpm runtime set node argon -g
```

> ​**重要提示**​：自 v11 起，通过 `pnpm runtime`​ 安装的 Node.js 将**不再包含**捆绑的 `npm`​、`npx`​ 和 `corepack`​。这可以显著减少安装时的文件数量、提升速度。如果需要，你仍然可以通过 `pnpm add -g npm`​ 来单独安装 `npm`。

## 📄 项目级配置：`devEngines.runtime`

为了让团队使用统一的 Node.js 版本，可以将运行时要求写入项目的 `package.json` 文件中。这比在命令行手动切换更可靠。

在 `package.json`​ 中添加 `devEngines.runtime` 字段。例如，声明你的项目需要 Node.js v24.4.0 或更高版本：

json

```
{
  "devEngines": {
    "runtime": {
      "name": "node",
      "version": "^24.4.0",
      "onFail": "download"
    }
  }
}
```

这里的配置项含义如下：

- ​**name**​: `"node"`​，表示声明的是 Node.js 运行时。也支持 `"deno"`​ 或 `"bun"`。
- ​**version**​: `"^24.4.0"`​，一个 [semver](https://semver.org/lang/zh-CN/) 版本范围。`pnpm install`​ 时会自动解析并安装符合该范围的最新版本，并将确切版本锁定在 `pnpm-lock.yaml` 中。
- ​**onFail**​: `"download"`​，定义了当本地环境不满足版本要求时的行为。可选值有 `"download"`​（自动下载）、`"warn"`​（仅警告）、`"error"`​（报错）或 `"ignore"`（忽略）。

配置后，只需运行 `pnpm install`，pnpm 就会根据配置自动处理 Node.js 的安装和校验。
