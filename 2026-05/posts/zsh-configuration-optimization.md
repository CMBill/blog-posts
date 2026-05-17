---
title: Zsh 配置与优化
published: 2026-05-15
description: Zsh 终端配置指南，包括安装、基本设置、插件管理器 Zinit 的使用，以及各种优化配置。
tags: [Zsh, 终端, Shell配置, Linux]
category: 系统配置
slug: zsh-configuration-optimization
draft: false
---

## 1 安装

```bash
sudo pacman -S zsh
```

## 2 更改默认终端

```bash
chsh -l # 查看安装了哪些 Shell
chsh -s /usr/bin/zsh # 修改当前账户的默认 Shell
```

## 3 切换到 Zsh 并配置

第一次启动时按照提示配置即可，也可以直接编辑`.zshrc`文件。

```zsh
# Lines configured by zsh-newuser-install
HISTFILE=~/.histfile
HISTSIZE=99999
SAVEHIST=99999
setopt autocd beep extendedglob nomatch notify
bindkey -v
# End of lines configured by zsh-newuser-install
# The following lines were added by compinstall
zstyle :compinstall filename '/home/bill/.zshrc'

autoload -Uz compinit
compinit
# End of lines added by compinstall

### Added by Zinit's installer
if [[ ! -f $HOME/.local/share/zinit/zinit.git/zinit.zsh ]]; then
    print -P "%F{33} %F{220}Installing %F{33}ZDHARMA-CONTINUUM%F{220} Initiative Plugin Manager (%F{33}zdharma-continuum/zinit%F{220})…%f"
    command mkdir -p "$HOME/.local/share/zinit" && command chmod g-rwX "$HOME/.local/share/zinit"
    command git clone https://github.com/zdharma-continuum/zinit "$HOME/.local/share/zinit/zinit.git" && \
        print -P "%F{33} %F{34}Installation successful.%f%b" || \
        print -P "%F{160} The clone has failed.%f%b"
fi

source "$HOME/.local/share/zinit/zinit.git/zinit.zsh"
autoload -Uz _zinit
(( ${+_comps} )) && _comps[zinit]=_zinit
### End of Zinit's installer chunk

# 绑定键盘上的特殊按键如End，可以命令行里按cat回车，再按下对应的按键即可查看
bindkey  "^[[H"   beginning-of-line
bindkey  "^[[F"   end-of-line
bindkey  "^[[3~"  delete-char

# 配置默认编辑器
export EDITOR='nvim'

# 配置插件
zinit light zsh-users/zsh-completions
zinit light zsh-users/zsh-autosuggestions
zinit light zdharma-continuum/fast-syntax-highlighting
zinit light agkozak/zsh-z

# Starship
eval "$(starship init zsh)"

# 终端语言
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8

# 别名
alias ls='ls --color=auto'
alias ll='ls -alhF'
alias grep='grep --color=auto'
```

### 3.1 基本 Zsh 设置

这一部分也是初始化的时候设置的。

- **历史记录**  
  ​`HISTFILE=~/.histfile`​：指定历史命令保存文件。  
  ​`HISTSIZE=99999`​ 和 `SAVEHIST=99999`：内存中及文件中的历史命令最大数量。
- **Shell 选项**  
  ​`setopt autocd beep extendedglob nomatch notify`：

  - ​`autocd`：直接输入目录名即可进入。
  - ​`beep`：出错时响铃。
  - ​`extendedglob`：启用扩展模式匹配。
  - ​`nomatch`：无匹配时不报错。
  - ​`notify`：后台任务完成时立即通知。
- **键绑定**  
  ​`bindkey -v`​：使用 `vi`​ 风格命令行编辑（默认是 `emacs` 风格）。

### 3.2 命令补全系统

通过 `compinit`​ 加载 Zsh 自带的智能补全，并设置 `compinstall` 保存补全配置。

### 3.3 插件管理器 Zinit

参考 [https://github.com/zdharma-continuum/zinit](https://github.com/zdharma-continuum/zinit)

- ​**自动安装**​：若未安装 Zinit，脚本会从 GitHub 克隆到 `~/.local/share/zinit`。
- ​**加载 Zinit**：然后加载其核心文件及补全定义。  
  Zinit 用于管理 Zsh 插件，轻量且高效。

### 3.4 键盘按键修正

针对终端发送的转义序列，绑定常用按键：

- ​`^[[H`​：`Home` → 行首
- ​`^[[F`​：`End` → 行尾
- ​`^[[3~`​：`Delete` → 删除光标后字符

### 3.5 编辑器环境变量

​`export EDITOR='nvim'`：设置 Neovim 为默认文本编辑器。

### 3.6 加载的 Zinit 插件

- ​`zsh-users/zsh-completions`：提供额外的补全定义。
- ​`zsh-users/zsh-autosuggestions`​：根据历史记录实时建议命令（按 `→` 补全）。
- ​`zdharma-continuum/fast-syntax-highlighting`​：高性能语法高亮，类似于`zsh-syntax-highlighting`。
- ​`agkozak/zsh-z`​：智能目录跳转，根据访问频率快速切换到常用目录，类似于`autojump`。

### 3.7 提示符 Starship

​`eval "$(starship init zsh)"`​：启用 [Starship](https://github.com/starship/starship) 跨 shell 提示符，显示 Git 状态、时间、路径等丰富信息。

### 3.8 终端语言环境

​`export LANG=zh_CN.UTF-8`​ 和 `export LC_ALL=zh_CN.UTF-8`：设置用户终端为中文 UTF-8 编码。

### 3.9 常用别名

- ​`ls='ls --color=auto'`：彩色显示目录内容。
- ​`ll='ls -alhF'`：列出所有文件（包括隐藏），人性化大小，并标记类型。
- ​`grep='grep --color=auto'`：匹配结果高亮显示。

‍
