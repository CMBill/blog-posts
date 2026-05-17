---
title: 常用的 colcon build 参数
published: 2026-05-17
description: colcon build 是 ROS 2 中用来构建工作空间的核心命令，本文按用途分类介绍最常用的参数。
tags: [ROS 2, colcon, 构建工具, AIGC]
category: 开发工具
slug: colcon-build-parameters
draft: false
---

​`colcon build` 是 ROS 2 中用来构建工作空间的核心命令。下面按用途分类，介绍最常用的一些参数。

---

### 1. 安装方式

- ​ **​`--symlink-install`​**​  
  使用符号链接安装文件，而不是复制。当你修改了 Python 脚本、配置文件或 launch 文件时，**无需重新构建**就能直接生效（对 Python 包、配置文件、launch 文件这类“改动频繁”的内容特别方便，但C++ 代码仍需重新编译）。开发调试时常用。
- ​ **​`--merge-install`​**​  
  将所有包安装到同一个 `install/`​ 目录下（类似 ROS 1 的结构），而不是默认的 `install/<pkg_name>`。某些场景下可以简化环境变量设置，不过通常推荐独立安装方式。

---

### 2. 包选择

- ​ **​`--packages-select`​**   
  只构建指定的几个包，不处理其他包。
-  **--packages-up-to**   
  构建指定包以及它的所有递归依赖包。适合你只关心某个包和它依赖的包链。
- ​ **​`--packages-above`​**   
  构建所有直接或间接依赖于指定包的包（反向依赖）。用于检查改动影响哪些下游包。
- ​ **​`--packages-skip`​**   
  跳过指定包不构建，但其他包仍正常构建。

---

### 3. 构建过程控制

- ​ **​`--continue-on-error`​**  
  即便某个包构建失败，也继续构建其他包，便于一次性发现问题包。
- ​ **​`--executor sequential`​**  
  强制顺序构建（默认会并行构建），方便调试依赖问题或排查编译顺序。
- ​ **​`--parallel-workers N`​**​  
  限制并行构建的任务数，避免资源紧张，例如 `--parallel-workers 4`。
- ​ **​`--cmake-args`​**​  
  将参数直接传递给 CMake，如：  
  ​`--cmake-args -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-O2"`
- ​ **​`--ament-cmake-args`​**​  
  将参数传递给 `ament_cmake` 构建系统（ROS 2 专用），用法同上。
- ​ **​`--cmake-clean-cache`​**  
  删除之前的 CMake 缓存，重新从头配置，相当于强制 clean 配置。
- ​ **​`--cmake-force-configure`​**  
  即使缓存存在，也强制重新运行 CMake 配置步骤。
- ​ **​`--allow-overriding`​** ​  
  改变某个包的构建类型（例如把一个原本用 `ament_cmake`​ 的包强制当成纯 `cmake` 包来构建），在混用不同构建系统时有用。

---

### 4. 输出与事件处理

- ​ **​`--event-handlers`​**  
  控制终端输出和通知行为。常用值：

  - ​`console_direct+` – 直接在终端实时打印所有包的编译输出（不缓冲）。
  - ​`console_cohesion+` – 每个包输出聚合显示（默认之一）。
  - ​`desktop_notification-`​ – 关闭桌面通知（构建完成/失败的通知）。  
    示例：`--event-handlers console_direct+`

---

### 5. 测试相关

- ​ **​`--build-tests`​**​  
  在构建过程中同时编译测试代码，后续才能运行 `colcon test`​。  
  也可以直接用 CMake 参数开启：`--cmake-args -DBUILD_TESTING=ON`。

---

### 6. 工作空间路径

- ​ **​`--build-base`​**​  
  指定构建中间文件存放目录，默认为工作空间下的 `build/`。
- ​ **​`--install-base`​** ​  
  指定最终安装文件的存放目录，默认为 `install/`。
- ​ **​`--paths`​**​  
  指定包含 ROS 包的源代码目录，默认为 `src/`，如果你的包放在别处可以用它指定。

---

### 7. 其他

- ​ **​`--metas`​**  
  使用一个元信息文件（meta file）来统一覆盖某些包的编译选项，适合复杂的混合工作空间。

---

### 常见场景示例

- **日常开发最常用**  
  ​`colcon build --symlink-install --packages-select my_package`
- **重新全量构建（清除缓存）**   
  ​`colcon build --cmake-clean-cache --cmake-force-configure`
- **只构建某个包及其依赖，并实时查看输出**  
  ​`colcon build --packages-up-to my_package --event-handlers console_direct+`
- **以 Release 模式编译**  
  ​`colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release`

掌握这些参数基本就能覆盖绝大部分开发和集成场景了。
