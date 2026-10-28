# Hugo + PaperMod 个人博客搭建指南

> 目标：从零搭建一个类似 [joshuapsteele.com](https://joshuapsteele.com) 的个人博客，支持全文搜索、深色模式、分类标签，并自动部署到 GitHub Pages。
> 技术栈：Hugo（Extended）+ PaperMod 主题 + GitHub Actions。

---

## 目录

1. [方案选择：为什么用静态博客](#1-方案选择为什么用静态博客)
2. [环境准备](#2-环境准备)
3. [创建站点与安装主题](#3-创建站点与安装主题)
4. [核心配置 hugo.yaml](#4-核心配置-hugoyaml)
5. [写作与本地预览](#5-写作与本地预览)
6. [部署到 GitHub Pages](#6-部署到-github-pages)
7. [日常写作流程](#7-日常写作流程)
8. [检查清单](#8-检查清单)
9. [本次实际执行记录](#9-本次实际执行记录2026-10-28)

---

## 1. 方案选择：为什么用静态博客

静态博客用静态生成器把 Markdown 转成 HTML，再托管到 GitHub Pages 等免费平台：**零成本、无需服务器、安全性高**，是技术圈最主流的个人博客方案。

### Hugo vs Hexo

| 对比项 | Hugo（Go） | Hexo（Node.js） |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 
| --- | --- | --- |
| 构建速度 | 极快，万篇文章不到 3 秒 | 文章量大（1000+ 篇）时明显变慢 |
| 模板语法 | Go 模板，对新手稍难 | EJS / Pug，上手快 |
| 中文生态 | 相对较少 | 教程、主题海量，中文资源最全 |
| 适合人群 | 追求性能、愿意读文档 | 刚入门、想快速出效果 |

本指南选择 **Hugo + PaperMod**：构建快、主题功能完整（搜索 / 深色模式 / 标签），部署链路简单。

---

## 2. 环境准备

需要安装 **Git** 和 **Hugo Extended（扩展版）**。PaperMod 部分功能依赖扩展版，务必不要装普通版。

- **Windows**：优先用 Scoop 安装

  ```powershell
  scoop install hugo-extended
  ```

  没有 Scoop 时可用 winget：`winget install --id Hugo.Hugo.Extended -e`。若从 GitHub 下载超时，可改用镜像下载便携版（本次实际采用，见第 9 节）。

- **macOS**：使用 Homebrew 安装

  ```bash
  brew install hugo
  ```

- 也可以从 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 下载对应系统的 `hugo_extended` 压缩包，解压后加入 PATH。

安装完成后验证：

```bash
hugo version   # 输出中应包含 "extended"
git --version
```

Git 需要先完成基础配置（用户名、邮箱）：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

---

## 3. 创建站点与安装主题

### 3.1 创建站点

```bash
hugo new site my-blog
cd my-blog
```

`my-blog` 可替换为你喜欢的目录名。若要在已有目录（如本项目的 `d:\study\blog`）中直接建站，目录非空会报错，需加 `--force`——它只补建缺失的骨架，不会删除已有文件：

```bash
hugo new site . --force
```

### 3.2 初始化 Git 仓库

```bash
git init
```

### 3.3 安装 PaperMod 主题

推荐使用 Git Submodule 方式，便于后续 `git submodule update --remote` 升级主题：

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

若 GitHub 直连超时，可先经镜像克隆，克隆完成后把 `.gitmodules` 里的 URL 改回官方地址（GitHub Actions 部署时仍能正常拉取）：

```bash
git submodule add --depth 1 https://gh-proxy.com/https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

目录结构大致如下：

```
my-blog/
├── archetypes/
├── content/
│   └── posts/          # 文章放在这里
├── static/
│   └── images/         # 图片统一放这里
├── themes/
│   └── PaperMod/
├── hugo.yaml           # 站点配置
└── .github/workflows/  # 部署工作流
```

---

## 4. 核心配置 hugo.yaml

站点根目录默认生成 `hugo.toml`。建议改名为 `hugo.yaml`（YAML 更清晰易读），然后填入以下基础配置，按注释修改为自己的信息：

```yaml
baseURL: "https://你的用户名.github.io/"
title: "你的博客标题"
locale: "zh-CN"                  # v0.158+ 用 locale，替代已弃用的 LanguageCode
defaultContentLanguage: "zh-cn"
theme: "PaperMod"

languages:
  zh-cn:
    label: "简体中文"
    locale: "zh-CN"
    weight: 1

# 首页额外输出 JSON，本地搜索索引依赖此项
outputs:
  home:
    - HTML
    - RSS
    - JSON

# 开启本地搜索（PaperMod 内置功能）
params:
  ShowCodeCopyButtons: true   # 显示代码复制按钮
  ShowReadingTime: true       # 显示阅读时间
  ShowShareButtons: true      # 显示分享按钮
  defaultTheme: "auto"        # 主题模式：light / dark / auto（跟随系统）
  ShowSearch: true            # 启用搜索

  homeInfoParams:
    Title: "你好，我是XXX"
    Content: >
      这里可以写一句你的个人简介或博客描述。

  # 社交图标（按需填写）
  socialIcons:
    - name: github
      url: "https://github.com/你的用户名"
    - name: twitter
      url: "https://twitter.com/你的用户名"
```

### 关键配置说明

| 配置项 | 作用 |
| --- | --- |
| `baseURL` | 站点最终访问地址，部署到 GitHub Pages 时必须与仓库名匹配 |
| `locale` / `defaultContentLanguage` | 站点语言。v0.158 起 `LanguageCode` 已弃用；两者需与 `languages.zh-cn` 配套，否则构建语言和页面 `<html lang>` 仍是 en |
| `theme: PaperMod` | 启用主题，路径需与 `themes/PaperMod` 一致 |
| `defaultTheme: "auto"` | 深色模式的开关。设为 `auto` 后跟随系统明暗自动切换，导航栏同时提供手动切换按钮 |
| `ShowSearch: true` | 启用内置本地搜索（基于 Fuse.js 生成索引），无需任何后端服务 |
| `outputs.home` 含 `JSON` | 搜索页依赖 `index.json`；缺少时搜索框会一直处于禁用状态 |
| `homeInfoParams` | 首页的标题与简介文案 |
| `socialIcons` | 导航栏社交图标，`name` 需使用 PaperMod 内置图标名 |

### 可选：导航菜单与分类标签

在配置中补充菜单（首页、文章、标签、归档等）：

```yaml
menu:
  main:
    - identifier: posts
      name: 文章
      url: /posts/
      weight: 10
    - identifier: archives
      name: 归档
      url: /archives/
      weight: 20
    - identifier: tags
      name: 标签
      url: /tags/
      weight: 30
    - identifier: search
      name: 搜索
      url: /search/
      weight: 40
```

归档页和搜索页不是自动生成的，需要两个单页文件指定布局，否则菜单会 404：

```markdown
<!-- content/archives.md -->
---
title: "归档"
layout: "archives"
url: "/archives/"
---

<!-- content/search.md -->
---
title: "搜索"
layout: "search"
url: "/search/"
placeholder: "输入关键词搜索文章"
---
```

分类标签无需额外安装插件：只要文章 front matter 里写了 `tags` / `categories`，PaperMod 就会自动生成 `/tags/`、`/categories/` 列表页，这也正是 joshuapsteele.com 的功能形态。

---

## 5. 写作与本地预览

### 5.1 新建文章

```bash
hugo new posts/我的第一篇文章.md
```

文件生成在 `content/posts/` 下，头部 `---` 之间是元数据，下方写正文：

```markdown
---
title: "我的第一篇文章"
date: 2026-10-28
draft: false
tags: ["Hugo", "博客"]
categories: ["教程"]
summary: "一句话摘要，会显示在文章列表里。"
---

正文内容，使用 Markdown 语法书写。
```

### 5.2 本地预览

```bash
hugo server -D
```

浏览器访问 <http://localhost:1313>，修改内容后页面会自动刷新。`-D` 表示同时渲染草稿（`draft: true`）文章。

### 5.3 图片管理

图片统一放在 `static/images/` 目录，文章中用绝对路径引用：

```markdown
![图片说明](/images/图片名.png)
```

构建时 `static/` 下的内容会原样复制到站点根目录，因此线上路径同样可用。

---

## 6. 部署到 GitHub Pages

这是实现「零成本 + 自动部署」的关键环节。

### 6.1 创建仓库

在 GitHub 创建新仓库，仓库名必须为 `你的用户名.github.io`（例如 `joshuapsteele.github.io`）。这是 GitHub Pages 为用户站点保留的命名格式，也是 `baseURL` 必须匹配的原因。

### 6.2 推送代码

```bash
git add .
git commit -m "init blog"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
git push -u origin main
```

### 6.3 配置 GitHub Pages

进入仓库 **Settings → Pages**，把 **Source** 从默认的 `Deploy from a branch` 改为 **`GitHub Actions`**。这样 GitHub 会使用 Actions 构建并发布站点，而不是读取某个分支的静态文件。

### 6.4 创建部署工作流

在项目根目录创建 `.github/workflows/hugo.yaml`：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive   # 必须拉取 PaperMod 子模块

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.169.0'  # 与本地版本一致；用 'latest' 可能被上游破坏性变更影响
          extended: true           # 主题依赖扩展版

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

工作流含义：每次推送到 `main` 分支时，自动检出代码（含子模块）→ 安装 Hugo 扩展版 → 构建到 `public/` → 上传产物 → 部署到 GitHub Pages。

配置完成后推送：

```bash
git add .github/workflows/hugo.yaml
git commit -m "add deploy workflow"
git push
```

几分钟后，博客即发布在 `https://你的用户名.github.io`。可在仓库 **Actions** 页签查看构建日志。

---

## 7. 日常写作流程

上线之后只需三步：

```bash
hugo new posts/新文章.md     # 1. 新建文章
hugo server -D              # 2. 本地预览
git add . && git commit -m "new post" && git push   # 3. 推送，自动发布
```

---

## 8. 检查清单

部署失败或页面样式异常时，按顺序排查：

- [ ] Hugo 是否为 **extended** 版本（`hugo version` 确认）
- [ ] `theme` 名称与 `themes/` 下目录名大小写一致
- [ ] `baseURL` 是否与仓库地址完全一致（含末尾 `/`）
- [ ] 仓库名是否为 `用户名.github.io`
- [ ] Settings → Pages 的 Source 是否已设为 **GitHub Actions**
- [ ] 工作流中是否设置了 `submodules: recursive`（否则主题缺失、构建报错）
- [ ] 工作流是否设置了 `extended: true`
- [ ] 文章 `draft: true` 时不会上线（`hugo --minify` 不渲染草稿）
- [ ] `outputs.home` 是否包含 `JSON`（缺了搜索框不可用）
- [ ] 是否用 `locale` + `defaultContentLanguage` 配置中文（新版 Hugo 中 `LanguageCode` 已弃用）
- [ ] 图片是否放在 `static/images/` 并用 `/images/xxx.png` 引用
- [ ] 仓库 Actions 权限是否允许部署（`pages: write`、`id-token: write`）

---
