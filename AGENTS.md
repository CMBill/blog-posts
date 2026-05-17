---
title: AGENTS.md
published: 2026-05-17
description: "Agent 说明"
draft: true
---

# AGENTS.md

## 仓库概述

这是 `CMBill.github.io` 博客的**文章内容仓库**，作为 Git 子模块（submodule）嵌入到主博客站点中。纯内容仓库，无构建/测试/代码。

## 目录结构

```
YYYY-MM/posts/    ← 博客文章按月份归档（如 2026-05/posts/xxx.md）
YYYY-MM/assets/   ← 文章相关资源（图片、数据文件等）
examples/         ← Firefly 主题示例文章（草稿状态，仅供参考）
```

## 文章格式

### 文件命名
- 使用英文，与文章的 Slug 一致，，如 `zsh-configuration-optimization.md`
- 支持 `.md`（Markdown）和 `.mdx`（MDX）两种格式

### Front-matter 规范

示例：

```yaml
---
title: Zsh 配置与优化
published: 2026-05-15
description: 详细的Zsh终端配置指南，包括安装、基本设置、插件管理器Zinit的使用，以及各种优化配置。
tags: [Zsh, 终端, Shell配置, Linux]
category: 系统配置
slug: zsh-configuration-optimization
draft: false
---
```

| 属性           | 描述                                                                                                                                                              |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `title`        | 文章标题。必填。                                                                                                                                                  |
| `published`    | 文章发布日期。必填。                                                                                                                                              |
| `updated`      | 文章更新日期。                                                                                                                                                    |
| `pinned`       | 是否将此文章置顶在文章列表顶部。                                                                                                                                  |
| `description`  | 文章的简短描述。显示在首页上。必填。                                                                                                                              |
| `image`        | 文章封面图片路径。<br/>1. 以 `http://` 或 `https://` 开头：使用网络图片<br/>2. 以 `/` 开头：`public` 目录中的图片<br/>3. 不带任何前缀：相对于 markdown 文件的路径 |
| `tags`         | 文章标签，使用 YAML 数组。必填。                                                                                                                                  |
| `category`     | 文章分类。必填。                                                                                                                                                  |
| `lang`         | 文章语言代码（如 `zh-CN`）。                                                                                                                                      |
| `licenseName`  | 文章内容的许可证名称。                                                                                                                                            |
| `licenseUrl`   | 文章内容的许可证链接。                                                                                                                                            |
| `author`       | 文章作者。                                                                                                                                                        |
| `sourceLink`   | 文章内容的来源链接或参考。                                                                                                                                        |
| `draft`        | 如果这篇文章仍是草稿，则不会显示。必填。                                                                                                                          |
| `comment`      | 是否启用此文章的评论功能。默认为 `true`。                                                                                                                         |
| `slug`         | 自定义文章 URL 路径。必填。                                                                                                                                       |
| `password`     | 文章密码。设置后文章内容将被 AES-256-GCM 加密，访客需输入密码才能查看。                                                                                           |
| `passwordHint` | 密码提示。                                                                                                                                                        |

- 除上面表格写明必填的属性外，其他属性均非必选，非必选属性不主动生成。
- `draft: true` 的文章不会显示在博客上
- 所有 `examples/` 中的文章均设为 `draft: true`

## 自定义文章 URL (Slug)

### 什么是 Slug？

Slug 是文章 URL 路径的自定义部分。如果不设置 slug，系统将使用文件名作为 URL。

### Slug 使用建议

1. **使用英文和连字符**：`my-awesome-post` 而不是 `my awesome post`
2. **保持简洁**：避免过长的 slug
3. **具有描述性**：让 URL 能够反映文章内容
4. **避免特殊字符**：只使用字母、数字和连字符
5. **保持一致性**：在整个博客中使用相似的命名模式

## 与主博客的关系

- 本仓库作为子模块挂载在 `CMBill/CMBill.github.io` 的 `src/content/posts/` 下
- push 到 `master` 分支后，CI 会自动触发主博客仓库的 `update-submodules.yml` workflow 以更新子模块指针
- 主分支是 `master`（不是 `main`）
- GitHub 仓库地址：`CMBill/blog-posts`
