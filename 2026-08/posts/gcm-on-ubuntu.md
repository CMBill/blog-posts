---
title: 在 Ubuntu 上使用 Git Credential Manager（GCM）实现认证
published: 2026-08-05
description: 在 Ubuntu 上从 GitHub Release 下载安装 Git Credential Manager（GCM），配置其使用 Seahorse（GNOME Keyring）作为凭证存储后端，实现访问私有仓库时通过浏览器完成登录认证。
tags: [Git, GCM, Ubuntu, 凭证管理, Linux, AIGC]
category: 系统配置
slug: gcm-on-ubuntu
draft: false
---

Git Credential Manager（GCM）是一个跨平台的凭证助手，能够安全地存储 Git 凭证并自动完成身份验证，同时支持 OAuth、双因素认证（2FA）等现代认证方式，且支持多账户场景，会根据不同的托管平台和仓库地址自动匹配对应的凭证。本文将介绍如何在 Ubuntu 系统上从 GitHub Release 下载安装 GCM，并配置其使用 Ubuntu 自带的 Seahorse（GNOME Keyring）作为凭证存储后端，最终实现访问私有仓库时通过浏览器完成登录认证的完整流程。

## 1 从 GitHub Release 下载安装 GCM

GCM 官方维护在 GitHub 上，项目地址为 [https://github.com/git-ecosystem/git-credential-manager](https://github.com/git-ecosystem/git-credential-manager)。Ubuntu/Debian 系统可以通过 `.deb` 包进行安装。

### 1.1 下载最新的 `.deb` 包

访问 GCM 的 [Releases 页面](https://github.com/git-ecosystem/git-credential-manager/releases)，找到最新版本的 `.deb` 包。通常文件名为 `gcm-linux-x64-{version}.deb`。

### 1.2 安装 `.deb` 包

记得更换为实际的路径。

```bash
sudo apt install ./path/to/gcm-linux-x64-{version}.deb
```

## 2 配置 GCM 使用 Seahorse 作为凭证存储

与 Windows 和 macOS 不同，Linux 系统没有默认的全局凭证存储机制，因此需要手动配置 GCM 使用哪种凭证存储后端。Ubuntu 系统自带 GNOME Keyring，可以通过 Seahorse（“密码与密钥”应用）进行图形化管理。

### 2.1 配置 credential.helper

将 GCM 设置为 Git 的凭证助手：

```bash
git config --global credential.helper manager
```

### 2.2 配置 credential.credentialStore 为 secretservice

GCM 在 Linux 上支持多种凭证存储方式，包括 `secretservice`（基于 freedesktop.org Secret Service API）、`gpg`（基于 GPG 加密）以及 `cache`（内存缓存）等。其中 `secretservice` 使用 `libsecret` 库通过 D-Bus 与 GNOME Keyring 或 KDE Wallet 等桌面凭证管理器交互，在 Ubuntu 上由 Seahorse（GNOME Keyring）提供支持。

执行以下命令将凭证存储设置为 `secretservice`：

```bash
git config --global credential.credentialStore secretservice
```

> 也可以通过环境变量方式配置：`export GCM_CREDENTIAL_STORE=secretservice`。

### 2.3 确保依赖已安装

`secretservice` 存储方式需要 `libsecret` 库的支持。在 Ubuntu 上通常已经预装，如果没有，可以手动安装：

```bash
sudo apt update
sudo apt install libsecret-1-0 libsecret-1-dev
```

### 2.4 重启系统

配置完成后，建议重启系统以确保所有更改生效。如果不重启，可能会遇到一些奇怪的错误。

```bash
sudo reboot
```

## 3 访问私有仓库——浏览器登录认证

完成安装和配置后，GCM 会在 Git 需要认证时自动介入。

### 3.1 首次认证流程

当你首次执行需要认证的 Git 操作（如 `git clone` 访问私有仓库、`git push` 等）时，GCM 会自动启动系统的默认浏览器，打开托管平台（如 GitHub）的授权页面。

例如，执行：

```bash
git clone https://github.com/your-username/private-repo.git
```

GCM 会弹出一个窗口，提示你使用浏览器登录。在浏览器中完成登录和授权后，凭证会被安全地存储在 GNOME Keyring 中，后续操作将自动使用已存储的凭证，无需再次输入。

### 3.2 查看已存储的凭证

Ubuntu 用户可以通过 Seahorse（“密码与密钥”应用）查看 GCM 存储的凭证：

1. 打开 Seahorse 应用（在应用菜单中搜索“密码与密钥”或 `seahorse`）
2. 在左侧面板中找到“Login”密钥环
3. 查找类似 `git:https://github.com/` 的条目

### 3.3 更新或删除凭证

如果需要更新凭证（例如更换了 GitHub 密码或 Personal Access Token），可以通过 Seahorse 删除旧凭证，然后在下一次 Git 操作时重新通过浏览器认证即可：

1. 打开 Seahorse
2. 在“登录”密钥环中找到对应的 Git 凭证条目
3. 右键删除
4. 下次执行 `git push` 或 `git clone` 时，GCM 会再次打开浏览器进行认证
