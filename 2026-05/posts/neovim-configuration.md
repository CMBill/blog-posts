---
title: Neovim 配置
published: 2026-05-17
description: Neovim 的配置指南，包括配置文件位置、将 vim 链接到 nvim 的方法，以及使用 lazy.nvim 管理插件的详细配置。
tags: [Neovim, Vim, 编辑器配置]
category: 系统配置
slug: neovim-configuration
draft: false
---

使用 Vim 的分支 [Neovim](https://neovim.io/)，与 Vim 不同，Neovim 的用户配置文件位于`$XDG_CONFIG_HOME/nvim/`​（默认是`~/.config/nvim/`​），全局配置文件（默认不存在）位于`$XDG_CONFIG_DIRS/nvim/sysinit.vim`​（默认是 `/etc/xdg/nvim/sysinit.vim`​），如果不存在，则会载入 `/usr/share/nvim/sysinit.vim`，后者不建议由用户编辑。同时，Neovim 的配置文件支持 Lua 语法。

Windows下配置文件位于`C:\Users\Bill\AppData\Local\nvim\init.vim`。

## 将 vim 链接到 nvim

在 Arch Linux 下，可以使用包[AUR (zh_CN) - neovim-symlinks](https://aur.archlinux.org/packages/neovim-symlinks)来办到。

在 Linux下，使用软链接：

```zsh
mkdir -p ~/.local/bin  # 创建用户级别的 bin 目录
ln -s $(which nvim) ~/.local/bin/vim  # 建立软链接
```

随后将自定义目录添加到 `PATH`​ 开头，在 `~/.bashrc`​（或 `~/.zshrc`）中添加：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

在 Powershell 中，使用别名：

单次配置：

```powershell
Set-Alias -Name vim -Value nvim
```

使用`echo $PROFILE`命令找到配置文件位置，随后在对应的目录下建立对应的配置文件，并在其中写入：

```ps
function vim {nvim}
```

## 配置文件及插件管理器

使用插件管理器[folke/lazy.nvim](https://github.com/folke/lazy.nvim)，通过直接编辑 Neovim 的配置文件即可引入。同时，有一个名为 [LazyVim/LazyVim](https://github.com/LazyVim/LazyVim) 的项目，提供了开箱即用的配置模板，功能强大，但是对于我而言不需要这么多功能，所以只选择了部分插件。Lazy nvim 直接通过编辑配置文件的方式引入，整个配置文件夹结构为：

```bash
nvim
├── init.lua
├── lazy-lock.json
└── lua
    ├── config
    │   └── lazy.lua
    └── plugins
        └── spec1.lua
```

具体的配置文件，可以参考我的 dotfile 仓库：[dotfiles/dotfiles/nvim at master · CMBill/dotfiles](https://github.com/CMBill/dotfiles/tree/master/dotfiles/nvim)

要安装 Lazy nvim，首先在`~/.config/nvim/init.lua`文件中引入：

```lua
require("config.lazy")
```

随后在配置文件夹下新建`lua/config`​文件夹，在`~/.config/nvim/lua/config/lazy.lua`中填写lazy.nvim启动文件：

```lua
-- Bootstrap lazy.nvim
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({ "git", "clone", "--filter=blob:none", "--branch=stable", lazyrepo, lazypath })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- Make sure to setup `mapleader` and `maplocalleader` before
-- loading lazy.nvim so that mappings are correct.
-- This is also a good place to setup other settings (vim.opt)
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Setup lazy.nvim
require("lazy").setup({
  spec = {
    -- import your plugins
    { import = "plugins" },
  },
  -- Configure any other settings here. See the documentation for more details.
  -- colorscheme that will be used when installing plugins.
  install = { colorscheme = { "habamax" } },
  -- automatically check for plugin updates
  checker = { enabled = true },
})
```

要安装、配置插件，则在`lua`​文件夹里新建`plugin`​文件夹，在其中的`lua`文件会被识别为插件配置。这里是我的插件配置：

1. **主题：catppuccin**  
   配置首先加载了 `catppuccin/nvim`​，并设置为高优先级（1000），确保界面尽早渲染。配色选用柔和的 `mocha` 风格，加载后直接应用，为整个编辑器定下温暖、护眼的基调。

2. **功能中心：snacks.nvim**  
   ​`folke/snacks.nvim` 是本配置的核心，它集成了十几项日常高频功能，且默认全部开启：大文件优化、仪表盘、文件浏览器、图片预览、缩进线、输入增强、通知系统、智能选择器、快速文件跳转、作用域高亮、平滑滚动、状态列和单词跳转等。通知超时设为 3 秒。

   快捷键设计非常有条理，几乎覆盖所有场景：

   - **查找与导航**：`<leader>ff`​ 查找文件，`<leader>fg`​ 查找 Git 文件，`<leader>/`​ 全文搜索，`<leader>fb`​ 切换缓冲区，`<leader>fp`​ 查找项目，`<leader>fr` 最近文件。
   - **Git 操作**：直接使用 `gs`​/`gl`​/`gd`​ 等组合查看状态、日志、差异、暂存等，`<leader>gg`​ 打开 Lazygit，`<leader>gB` 在浏览器中打开远程仓库。
   - **LSP 集成**：`gd`​ 跳转定义，`gr`​ 查看引用，`gy`​ 类型定义，`<leader>ss` 工作区符号等，完全取代原生 LSP 浮窗，界面更统一。
   - **实用工具**：`<leader>e`​ 打开文件浏览器，`<leader>z`​ 进入禅模式，`<leader>.`​ 打开草稿缓冲区，`<c-/>`​ 切换终端，`<leader>bd`​ 删除缓冲区，`<leader>cR` 重命名文件。

   ​`init`​ 函数中还准备了调试辅助（`dd`​、`bt`​）和一系列 `toggle`​ 映射，比如 `<leader>us`​ 切换拼写检查、`<leader>ul`​ 切换行号、`<leader>ud` 切换诊断显示等，一键控制编辑器状态，非常方便。
3. **按键提示：which-key.nvim**  
   配置了 `folke/which-key.nvim`​，采用 `helix`​ 风格预设，在按下 `<leader>`​ 后自动弹出可用的快捷键菜单。额外添加了 `<leader>?` 手动显示当前缓冲区专属快捷键，帮助记忆不常用的组合键。
4. **状态栏：lualine.nvim**  
   ​`nvim-lualine/lualine.nvim`​ 在 `VeryLazy`​ 时加载，直接使用 `catppuccin-mocha` 主题，使状态栏与整体配色完美融合，无需额外调校。
5. **图标与配对：mini.nvim 生态**  
   选取了两个轻量的 mini 插件：`mini.icons`​ 为各类文件类型提供图标支持，按需延迟加载；`mini.pairs` 在进入插入模式时自动配对括号、引号等，保持编辑体验流畅。
6. **Git 装饰：gitsigns.nvim**  
   ​`lewis6991/gitsigns.nvim`​ 在打开文件时即生效，用自定义符号（如 `┃`​ 表示新增或修改，`_` 表示删除）在左侧符号列标示变更状态，且区分暂存与未暂存的符号。配置还启用了当前行 blame 预览（行尾显示作者和时间），延迟 1 秒出现，同时限制超长文件禁用，兼顾性能与信息展示。

```lua
return {
    {
        "catppuccin/nvim",
        name = "catppuccin",
        priority = 1000,
        config = function()
            require("catppuccin").setup({
                flavour = "mocha",
            })
            vim.cmd.colorscheme "catppuccin"
        end
    },
    {
        "folke/snacks.nvim",
        priority = 1000,
        lazy = false,
        ---@type snacks.Config
        opts = {
            bigfile = { enabled = true },
            dashboard = { enabled = true },
            explorer = { enabled = true },
            image = { enabled = true },
            indent = { enabled = true },
            input = { enabled = true },
            notifier = {
                enabled = true,
                timeout = 3000,
            },
            picker = { enabled = true },
            quickfile = { enabled = true },
            scope = { enabled = true },
            scroll = { enabled = true },
            statuscolumn = { enabled = true },
            words = { enabled = true },
            styles = {
                notification = {
                    -- wo = { wrap = true } -- Wrap notifications
                }
            }
        },
        keys = {
            -- Top Pickers & Explorer
            { "<leader><space>", function() Snacks.picker.smart() end, desc = "智能查找文件" },
            { "<leader>,", function() Snacks.picker.buffers() end, desc = "缓冲区" },
            { "<leader>/", function() Snacks.picker.grep() end, desc = "全文搜索" },
            { "<leader>:", function() Snacks.picker.command_history() end, desc = "命令历史" },
            { "<leader>n", function() Snacks.picker.notifications() end, desc = "通知历史" },
            { "<leader>e", function() Snacks.explorer() end, desc = "文件浏览器" },
            -- find
            { "<leader>fb", function() Snacks.picker.buffers() end, desc = "缓冲区" },
            { "<leader>fc", function() Snacks.picker.files({ cwd = vim.fn.stdpath("config") }) end, desc = "查找配置文件" },
            { "<leader>ff", function() Snacks.picker.files() end, desc = "查找文件" },
            { "<leader>fg", function() Snacks.picker.git_files() end, desc = "查找 Git 文件" },
            { "<leader>fp", function() Snacks.picker.projects() end, desc = "项目" },
            { "<leader>fr", function() Snacks.picker.recent() end, desc = "最近" },
            -- git
            { "<leader>gb", function() Snacks.picker.git_branches() end, desc = "Git Branches" },
            { "<leader>gl", function() Snacks.picker.git_log() end, desc = "Git Log" },
            { "<leader>gL", function() Snacks.picker.git_log_line() end, desc = "Git Log Line" },
            { "<leader>gs", function() Snacks.picker.git_status() end, desc = "Git Status" },
            { "<leader>gS", function() Snacks.picker.git_stash() end, desc = "Git Stash" },
            { "<leader>gd", function() Snacks.picker.git_diff() end, desc = "Git Diff (Hunks)" },
            { "<leader>gf", function() Snacks.picker.git_log_file() end, desc = "Git Log File" },
            -- gh
            { "<leader>gi", function() Snacks.picker.gh_issue() end, desc = "GitHub Issues (open)" },
            { "<leader>gI", function() Snacks.picker.gh_issue({ state = "all" }) end, desc = "GitHub Issues
(all)" },
            { "<leader>gp", function() Snacks.picker.gh_pr() end, desc = "GitHub Pull Requests (open)" },
            { "<leader>gP", function() Snacks.picker.gh_pr({ state = "all" }) end, desc = "GitHub Pull Requests (all)" },
            -- Grep
            { "<leader>sb", function() Snacks.picker.lines() end, desc = "当前缓冲区行" },
            { "<leader>sB", function() Snacks.picker.grep_buffers() end, desc = "搜索已打开缓冲区" },
            { "<leader>sg", function() Snacks.picker.grep() end, desc = "搜索文本" },
            { "<leader>sw", function() Snacks.picker.grep_word() end, desc = "选区或单词", mode = { "n", "x" } },
            -- search
            { '<leader>s"', function() Snacks.picker.registers() end, desc = "Registers" },
            { '<leader>s/', function() Snacks.picker.search_history() end, desc = "搜索历史" },
            { "<leader>sa", function() Snacks.picker.autocmds() end, desc = "自动命令" },
            { "<leader>sb", function() Snacks.picker.lines() end, desc = "当前缓冲区行" },
            { "<leader>sc", function() Snacks.picker.command_history() end, desc = "命令历史" },
            { "<leader>sC", function() Snacks.picker.commands() end, desc = "命令" },
            { "<leader>sd", function() Snacks.picker.diagnostics() end, desc = "诊断" },
            { "<leader>sD", function() Snacks.picker.diagnostics_buffer() end, desc = "缓冲区诊断" },
            { "<leader>sh", function() Snacks.picker.help() end, desc = "帮助页面" },
            { "<leader>sH", function() Snacks.picker.highlights() end, desc = "高亮组" },
            { "<leader>si", function() Snacks.picker.icons() end, desc = "图标" },
            { "<leader>sj", function() Snacks.picker.jumps() end, desc = "跳转列表" },
            { "<leader>sk", function() Snacks.picker.keymaps() end, desc = "按键映射" },
            { "<leader>sl", function() Snacks.picker.loclist() end, desc = "位置列表" },
            { "<leader>sm", function() Snacks.picker.marks() end, desc = "标记" },
            { "<leader>sM", function() Snacks.picker.man() end, desc = "手册" },
            { "<leader>sp", function() Snacks.picker.lazy() end, desc = "搜索插件配置" },
            { "<leader>sq", function() Snacks.picker.qflist() end, desc = "快速修复列表" },
            { "<leader>sR", function() Snacks.picker.resume() end, desc = "恢复" },
            { "<leader>su", function() Snacks.picker.undo() end, desc = "撤销历史" },
            { "<leader>uC", function() Snacks.picker.colorschemes() end, desc = "配色方案" },
            -- LSP
            { "gd", function() Snacks.picker.lsp_definitions() end, desc = "转到定义" },
            { "gD", function() Snacks.picker.lsp_declarations() end, desc = "转到声明" },
            { "gr", function() Snacks.picker.lsp_references() end, nowait = true, desc = "引用" },
            { "gI", function() Snacks.picker.lsp_implementations() end, desc = "转到实现" },
            { "gy", function() Snacks.picker.lsp_type_definitions() end, desc = "转到类型定义" },
            { "gai", function() Snacks.picker.lsp_incoming_calls() end, desc = "传入调用" },
            { "gao", function() Snacks.picker.lsp_outgoing_calls() end, desc = "传出调用" },
            { "<leader>ss", function() Snacks.picker.lsp_symbols() end, desc = "LSP 符号" },
            { "<leader>sS", function() Snacks.picker.lsp_workspace_symbols() end, desc = "LSP Workspace 符号" },
            -- Other
            { "<leader>z", function() Snacks.zen() end, desc = "切换 Zen Mode" },
            { "<leader>Z", function() Snacks.zen.zoom() end, desc = "切换放大模式" },
            { "<leader>.", function() Snacks.scratch() end, desc = "切换临时缓冲区" },
            { "<leader>S", function() Snacks.scratch.select() end, desc = "选择临时缓冲区" },
            { "<leader>n", function() Snacks.notifier.show_history() end, desc = "通知历史" },
            { "<leader>bd", function() Snacks.bufdelete() end, desc = "删除缓冲区" },
            { "<leader>cR", function() Snacks.rename.rename_file() end, desc = "重命名文件" },
            { "<leader>gB", function() Snacks.gitbrowse() end, desc = "Git 浏览", mode = { "n", "v" } },
            { "<leader>gg", function() Snacks.lazygit() end, desc = "Lazygit" },
            { "<leader>un", function() Snacks.notifier.hide() end, desc = "清除所有通知" },
            { "<c-/>", function() Snacks.terminal() end, desc = "切换终端" },
            { "<c-_>", function() Snacks.terminal() end, desc = "which_key_ignore" },
            { "]]", function() Snacks.words.jump(vim.v.count1) end, desc = "下一个引用", mode = { "n", "t" } },
            { "[[", function() Snacks.words.jump(-vim.v.count1) end, desc = "上一个引用", mode = { "n", "t"
} },
            {
                "<leader>N",
                desc = "Neovim News",
                function()
                    Snacks.win({
                        file = vim.api.nvim_get_runtime_file("doc/news.txt", false)[1],
                        width = 0.6,
                        height = 0.6,
                        wo = {
                            spell = false,
                            wrap = false,
                            signcolumn = "yes",
                            statuscolumn = " ",
                            conceallevel = 3,
                        },
                    })
                end,
            }
        },
        init = function()
            vim.api.nvim_create_autocmd("User", {
                pattern = "VeryLazy",
                callback = function()
                    -- Setup some globals for debugging (lazy-loaded)
                    _G.dd = function(...)
                        Snacks.debug.inspect(...)
                    end
                    _G.bt = function()
                        Snacks.debug.backtrace()
                    end

                    -- Override print to use snacks for `:=` command
                    if vim.fn.has("nvim-0.11") == 1 then
                        vim._print = function(_, ...)
                            dd(...)
                        end
                    else
                        vim.print = _G.dd
                    end

                    -- Create some toggle mappings
                    Snacks.toggle.option("spell", { name = "Spelling" }):map("<leader>us")
                    Snacks.toggle.option("wrap", { name = "Wrap" }):map("<leader>uw")
                    Snacks.toggle.option("relativenumber", { name = "Relative Number" }):map("<leader>uL")
                    Snacks.toggle.diagnostics():map("<leader>ud")
                    Snacks.toggle.line_number():map("<leader>ul")
                    Snacks.toggle.option("conceallevel",
                        { off = 0, on = vim.o.conceallevel > 0 and vim.o.conceallevel or 2 }):map("<leader>uc")
                    Snacks.toggle.treesitter():map("<leader>uT")
                    Snacks.toggle.option("background", { off = "light", on = "dark", name = "Dark Background" }):map(
                        "<leader>ub")
                    Snacks.toggle.inlay_hints():map("<leader>uh")
                    Snacks.toggle.indent():map("<leader>ug")
                    Snacks.toggle.dim():map("<leader>uD")
                end,
            })
        end,
    },
    {
        "folke/which-key.nvim",
        event = "VeryLazy",
        opts = {
            preset = "helix",
            spec = {},
        },
        keys = {
            {
                "<leader>?",
                function()
                    require("which-key").show({ global = false })
                end,
                desc = "当前缓冲区按键提示 (which-key)",
            },
        },
    },
    {
        'nvim-lualine/lualine.nvim',
        event = "VeryLazy",
        dependencies = { 'nvim-tree/nvim-web-devicons' },
        opts = {
            theme = 'catppuccin-mocha',
        }
    },
    {
        'nvim-mini/mini.icons',
        lazy = true,
        version = false
    },
    {
        'nvim-mini/mini.pairs',
        event = "InsertEnter",
        opts = {},
        version = false
    },
    {
        "lewis6991/gitsigns.nvim",
        event = { "BufReadPre", "BufNewFile" },
        opts = {
            signs                        = {
                add          = { text = '┃' },
                change       = { text = '┃' },
                delete       = { text = '_' },
                topdelete    = { text = '‾' },
                changedelete = { text = '~' },
                untracked    = { text = '┆' },
            },
            signs_staged                 = {
                add          = { text = '┃' },
                change       = { text = '┃' },
                delete       = { text = '_' },
                topdelete    = { text = '‾' },
                changedelete = { text = '~' },
                untracked    = { text = '┆' },
            },
            signs_staged_enable          = true,
            signcolumn                   = true, -- Toggle with `:Gitsigns toggle_signs`
            numhl                        = false, -- Toggle with `:Gitsigns toggle_numhl`
            linehl                       = false, -- Toggle with `:Gitsigns toggle_linehl`
            word_diff                    = false, -- Toggle with `:Gitsigns toggle_word_diff`
            watch_gitdir                 = {
                follow_files = true
            },
            auto_attach                  = true,
            attach_to_untracked          = false,
            current_line_blame           = false, -- Toggle with `:Gitsigns toggle_current_line_blame`
            current_line_blame_opts      = {
                virt_text = true,
                virt_text_pos = 'eol', -- 'eol' | 'overlay' | 'right_align'
                delay = 1000,
                ignore_whitespace = false,
                virt_text_priority = 100,
                use_focus = true,
            },
            current_line_blame_formatter = '<author>, <author_time:%R> - <summary>',
            blame_formatter              = nil, -- Use default
            sign_priority                = 6,
            update_debounce              = 100,
            status_formatter             = nil, -- Use default
            max_file_length              = 40000, -- Disable if file is longer than this (in lines)
            preview_config               = {
                -- Options passed to nvim_open_win
                style = 'minimal',
                relative = 'cursor',
                row = 0,
                col = 1
            },
        }
    }

}
```
