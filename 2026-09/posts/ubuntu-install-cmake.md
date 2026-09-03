---
title: 在 Ubuntu 上安装更高版本的 CMake
published: 2026-09-03
description: 在较老的 Ubuntu 版本上通过添加 Kitware 官方 APT 仓库安装更高版本 CMake 的完整指南，涵盖安装依赖、获取 GPG 签名密钥、按发行版添加仓库、安装 keyring 及 CMake 等步骤。
tags: [CMake, Ubuntu, Linux, 构建工具, 包管理]
category: 系统配置
slug: ubuntu-install-cmake
draft: false
---

参考：[Kitware APT Repository](https://apt.kitware.com/)

对于较老的 Ubuntu 版本，其包管理器中提供的 CMake 往往也是很老的版本，无法编译某些要求更高的程序。可以通过添加 CMake 官方提供的 Kitware APT Repository 来安装更高版本的 CMake。

## 1 安装依赖

```bash
sudo apt-get update
sudo apt-get install ca-certificates gpg wget
```

## 2 获取签名密钥

```bash
test -f /usr/share/doc/kitware-archive-keyring/copyright ||
wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | sudo tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null
```

## 3 添加仓库

Ubuntu Resolute Raccoon (26.04):

```bash
echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ resolute-rc main' | sudo tee -a /etc/apt/sources.list.d/kitware.list >/dev/null
sudo apt-get update
```

Ubuntu Noble Numbat (24.04):

```bash
echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ noble main' | sudo tee /etc/apt/sources.list.d/kitware.list >/dev/null
sudo apt-get update
```

Ubuntu Jammy Jellyfish (22.04):

```bash
echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ jammy main' | sudo tee /etc/apt/sources.list.d/kitware.list >/dev/null
sudo apt-get update
```

Ubuntu Focal Fossa (20.04):

```bash
echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ focal main' | sudo tee /etc/apt/sources.list.d/kitware.list >/dev/null
sudo apt-get update
```

## 4 安装 keyring

```bash
sudo apt-get install kitware-archive-keyring
```

## 5 安装 CMake

```bash
sudo apt-get install cmake
```
