---
title: "用 Hugo 搭建个人博客"
date: 2026-10-28
draft: false
tags: ["Hugo", "PaperMod", "博客"]
categories: ["教程"]
summary: "记录这个博客的搭建过程：Hugo + PaperMod + GitHub Pages。"
---

这是博客的第一篇文章，用来验证主题、搜索、标签和深色模式是否正常工作。

## 搭建过程

1. 安装 Hugo Extended 与 Git
2. 用 `hugo new site` 创建站点，安装 PaperMod 主题
3. 配置 `hugo.yaml`
4. 推送代码，由 GitHub Actions 自动部署

## 代码块测试

```bash
hugo server -D
```

如果右上角有复制按钮、页面能切换明暗主题、搜索页能搜到这篇文章，说明配置已生效。
