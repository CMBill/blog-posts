---
title: Ubuntu 安装 nVidia 相关
published: 2026-07-13
description: Ubuntu 系统下安装 nVidia 显卡驱动、CUDA Toolkit 和 cuDNN 的完整指南，包含禁用 nouveau、添加官方存储库等步骤。
tags: [nVidia, CUDA, cuDNN, Ubuntu, 显卡驱动, GPU]
category: 系统配置
slug: ubuntu-install-nvidia
draft: false
---

参考：[CUDA Toolkit Downloads](https://developer.nvidia.com/cuda-downloads)、[ubuntu下cuda-keyring_1.1-1_all.deb包作用](https://blog.mvpbang.com/p/caa2e4a241564356b11e038e3da408fc/)

## 1 安装 CUDA 与显卡驱动

### 1.1 关闭系统自带驱动 nouveau

先通过命令`lsmod | grep nouveau`查看 nouveau 驱动的启用情况，若无输出则表示已经禁用。

要禁用 nouveau 驱动，编辑`/etc/modprobe.d/blacklist.conf`文件，在末尾加上这两行并保存：

```conf
blacklist nouveau
options nouveau modeset=0
```

随后在终端中：

```bash
sudo update-initramfs -u 
```

然后重启即可。

### 1.2 添加 NVIDIA 存储库

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
```

## 2 安装显卡驱动

```bash
sudo apt-get install -y nvidia-open # 会安装最新版本的显卡驱动
# 或者指定版本
# sudo apt-get install -y nvidia-open-580
```

> [!TIP]
> Ubuntu 22.04系统安装时可能会编译报错，是由于不支持系统默认安装的GCC 11版本，手动安装 GCC 12 即可。
>
> ```bash
> sudo apt install gcc-12
> ```

安装完成后，运行`nvidia-smi`即可检查是否安装成功。

对于大部分情况，安装好显卡驱动即可，不需执行后续操作。

## 3 安装 CUDA Toolkit

安装好显卡驱动后，即已经完成 CUDA 相关运行时的安装。

最新版本的 CUDA Toolkit 可以参考 [CUDA Toolkit Downloads](https://developer.nvidia.com/cuda-downloads)，对于 Ubuntu 20.04 最高支持到 CUDA 12.9，因此参考[CUDA Toolkit 12.9 Downloads](https://developer.nvidia.com/cuda-12-9-0-download-archive?target_os=Linux&amp;target_arch=x86_64&amp;Distribution=Ubuntu&amp;target_version=20.04&amp;target_type=deb_network)。在前面已经添加好存储库后直接运行：

```bash
sudo apt-get -y install cuda-toolkit-13-3
```

随后编辑`~/.bashrc`配置环境变量。

```bash
export CUDA_HOME=/usr/local/cuda
export PATH=$PATH:$CUDA_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CUDA_HOME/lib64
export LIBRARY_PATH=$LIBRARY_PATH:$CUDA_HOME/lib64
```

重启终端后运行`nvcc -V`查看安装信息。

## 4 安装 cuDNN

最新版本的 cuDNN 可以参考 [cuDNN Downloads](https://developer.nvidia.com/cudnn-downloads)，对于Ubuntu 20.04 最高支持到 cuDNN 9.10.2，因此参考[cuDNN 9.10.2 Downloads](https://developer.nvidia.com/cudnn-9-10-2-download-archive?target_os=Linux&amp;target_arch=x86_64&amp;Distribution=Ubuntu&amp;target_version=20.04&amp;target_type=deb_network)。在前面已经添加好存储库后直接运行：

```bash
# 对于 CUDA 12，执行
sudo apt-get -y install cudnn-cuda-12
# 对于 CUDA 13，执行
sudo apt-get -y install cudnn-cuda-13
```