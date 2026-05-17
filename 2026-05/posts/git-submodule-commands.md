---
title: Git Submodule 命令
published: 2026-05-17
description: Git Submodule 常用命令与配置详解，包括添加、克隆、更新、删除等操作及配置文件说明。
tags: [Git, Submodule, 版本控制, AIGC]
category: 开发工具
slug: git-submodule-commands
draft: false
---

​`git submodule`是用于在一个仓库中嵌套管理另一个独立 Git 仓库的工具。主仓库仅记录子模块所处的具体提交（commit），而不直接追踪子模块中的文件内容。

## 常用命令

### 1. 添加子模块

```bash
git submodule add <仓库URL> <本地路径>
```

执行后会生成 `.gitmodules` 文件，并记录子模块当前所在的 commit。

### 2. 克隆含有子模块的仓库

- 一步到位克隆：

  ```bash
  git clone --recurse-submodules <主仓库URL>
  ```
- 已克隆但子模块为空时，手动初始化并更新：

  ```bash
  git submodule update --init --recursive
  ```

此处的`--init`​参数相当于先执行`init`​子命令后再执行`update`​子命令，`git submodule init`​会根据 `.gitmodules`​ 文件的信息，初始化本地 Git 配置（`.git/config`），

### 3. 更新子模块

- 将子模块更新到远程仓库的最新提交：

  ```bash
  git submodule update --remote
  ```
- 更新后，主仓库中记录的子模块指针会变化，需要手动提交：

  ```bash
  git add <子模块路径>
  git commit -m "更新子模块到最新版本"
  ```

### 4. 查看子模块状态

```bash
git submodule status
```

输出中前缀含义：

- ​`-` 表示未初始化
- ​`+` 表示当前检出的提交与主仓库记录的提交不一致

### 5. 删除子模块

```bash
git submodule deinit <路径>
git rm <路径>
rm -rf .git/modules/<路径>
```

最后提交删除操作。

### 6. 同步子模块 URL

​`sync`​子命令，它的作用是将`.gitmodules`​文件中定义的远程 URL 同步到本地 Git 配置（`.git/config`）中。

#### 何时需要

当子模块的远程仓库地址发生变更（例如迁移到新服务器）时，仅修改 `.gitmodules`​ 是不够的。本地的 `.git/config`​ 仍保留旧地址，导致 `git submodule update`​ 等命令访问错误。此时需运行 `sync`​ 使本地配置与 `.gitmodules` 一致。

#### 典型场景

1. 子模块远程库从 `github.com/old/repo`​ 迁至 `gitlab.com/new/repo`。
2. 更新 `.gitmodules` 中的 URL 并提交。
3. 其他成员拉取主仓库后，执行 `git submodule sync`，否则更新时会尝试从旧地址拉取而失败。

#### 用法

```bash
# 同步所有子模块
git submodule sync

# 同步指定子模块
git submodule sync <子模块路径>

# 递归同步（子模块嵌套时）
git submodule sync --recursive
```

## 子模块在仓库中的配置文件

### 1. `.gitmodules` —— 版本控制的配置文件

位于主仓库根目录，采用 INI 格式，记录了子模块的逻辑路径和远程 URL。**该文件随主仓库一起版本控制**，他人克隆后可见。

```ini
[submodule "libs/mylib"]
    path = libs/mylib
    url = https://github.com/example/lib.git
[submodule "vendor/other"]
    path = vendor/other
    url = git@gitlab.com:team/other.git
    branch = develop
```

- ​`path`：子模块在主仓库中的相对路径。
- ​`url`：子模块的远程仓库地址。
- 可选字段如 `branch`​，指定 `git submodule update --remote` 时跟踪的分支（不写则默认跟踪主仓库记录的 commit）。

### 2. `.git/config` —— 本地仓库配置（不版本控制）

执行 `git submodule init`​ 后，`.gitmodules`​ 中的配置会被复制到 `.git/config` 中，形成类似结构：

```ini
[submodule "libs/mylib"]
    url = https://github.com/example/lib.git
    active = true
```

- 后续 `git submodule update`​ 等命令实际读取的是此处的 `url`。
- 若 `.gitmodules`​ 中的 URL 变化，需手动执行 `git submodule sync` 来同步此处。
- 该文件**仅在本地存在，不会被提交**，每个人的副本独立。

### 3. 主仓库的树对象 —— 子模块指针的存储

子模块本质上是一个类型为 commit 的 Git 对象，它在主仓库每一次提交的树对象中表现为一个特殊条目：

- 模式：`160000`​（普通文件是 `100644`​，目录是 `040000`）。
- 名称：子模块的目录名。
- 哈希：指向子模块仓库中某一个具体 commit 的 SHA-1 值。

用底层命令可查看到类似：

```bash
$ git ls-tree HEAD libs/mylib
160000 commit 1a2b3c4d...  libs/mylib
```

这个 commit SHA 决定了子模块检出到哪个版本。当你在主仓库中 `git add libs/mylib` 并提交时，实际上就是更新了这个哈希值。

### 4. `.git/modules/<路径>` —— 子模块实际的 Git 数据

子模块本身的仓库（对象库、引用等）存储在主仓库的 `.git/modules/<子模块路径>`​ 目录下（旧版本 Git 直接存放在子模块目录内的 `.git` 文件中）。  
这个目录对用户一般是透明的，但它是子模块能独立进行 Git 操作的基础。

## 一些操作

### 将子文件夹独立为子模块

假设原仓库结构：

```
my-project/
├── .git/
├── target-folder/       ← 想独立为子模块的文件夹
│   ├── a.txt
│   └── b.js
└── other-stuff/
```

最终：`target-folder`​ 变成一个独立的 Git 仓库，并在原仓库中作为子模块引用，且所有历史提交（仅针对 `target-folder/` 的修改）都保留在新仓库里。

#### 保留子文件夹提交历史

1. 提取文件夹的历史到一个新分支

   进入原仓库根目录，运行：

   ```bash
   git subtree split --prefix=target-folder --branch=subrepo-branch
   ```

   这会创建一个名为 `subrepo-branch`​ 的新分支，其中只包含 `target-folder/` 的历史记录（就像这个文件夹一直是独立仓库一样）。
2. 创建一个新的 Git 仓库并推送

   ```bash
   # 创建临时目录并初始化新仓库
   mkdir /tmp/new-subrepo
   cd /tmp/new-subrepo
   git init

   # 拉取刚才 split 出来的历史
   git pull /path/to/my-project subrepo-branch

   # 添加远程仓库（例如 GitHub 上新创建的空仓库）
   git remote add origin <新仓库的远程地址>
   git push -u origin master
   ```
3. 回到原仓库，删除原文件夹并添加子模块

   ```bash
   cd /path/to/my-project

   # 删除原文件夹（注意先确保新仓库已推送完成）
   git rm -rf target-folder
   git commit -m "Remove target-folder to replace with submodule"

   # 添加子模块，使用新仓库的地址
   git submodule add <新仓库的远程地址> target-folder

   # 提交子模块配置
   git commit -m "Add target-folder as submodule"
   ```
4. 清理临时分支

   ```bash
   git branch -D subrepo-branch
   ```

#### 不保留子文件夹提交历史

如果你不需要保留 `target-folder` 在原仓库中的提交历史，可以直接：

1. 把 `target-folder` 的内容复制到一个全新的 Git 仓库并推送。
2. 在原仓库中删除 `target-folder`​（`git rm -rf target-folder`）。
3. ​`git submodule add <新仓库地址> target-folder`。
4. 提交。

历史会丢失（新仓库只有一次初始提交），但操作更快。
