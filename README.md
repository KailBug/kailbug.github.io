# kailbug.github.io

KailBug 的个人技术主页、Agent Engineering 知识库与开源项目展示页。

在线访问：<https://kailbug.github.io/>

## Tech Stack

- Astro
- TypeScript
- Markdown / MDX
- pnpm
- GitHub Actions
- GitHub Pages

## Local Development

环境要求：

- Node.js >= 22.12
- pnpm 11
- Git

克隆并启动项目：

```bash
git clone https://github.com/KailBug/kailbug.github.io.git
cd kailbug.github.io
pnpm install
pnpm dev
```

本地开发地址通常为：

```text
http://localhost:4321/
```

## Commands

| Command | Description |
| --- | --- |
| `pnpm dev` | 启动本地开发服务器 |
| `pnpm build` | 构建生产版本到 `dist/` |
| `pnpm preview` | 本地预览生产构建 |
| `pnpm astro` | 运行 Astro CLI |

## Writing an Article

文章保存在：

```text
src/content/blog/
```

创建新的 Markdown 或 MDX 文件：

```text
src/content/blog/new-article.md
```

Frontmatter 示例：

```yaml
---
title: "Article title"
description: "A short article description."
date: 2026-08-23
category: "Agent Runtime"
tags:
  - "Agent"
  - "Runtime"
draft: false
featured: false
---
```

字段说明：

- `title`：文章标题。
- `description`：文章摘要。
- `date`：发布日期。
- `updatedDate`：可选的更新时间。
- `category`：文章分类。
- `tags`：文章标签。
- `draft`：为 `true` 时不会生成线上文章页面。
- `featured`：为 `true` 时可以显示在首页 Featured Articles。

普通文章优先使用 Markdown。只有需要在正文中引入 Astro 组件或表达式时才使用 MDX。

> 仓库是公开的。`draft: true` 只会阻止文章部署到网站，不会隐藏仓库中的 Markdown 源文件。

## Publishing Workflow

开始修改前，先同步远程分支并启动开发服务器：

```bash
git pull
pnpm dev
```

完成文章或网站修改后：

```bash
git add .
git commit -m "article: add new article"
git push
```

推送到 `main` 后：

```text
GitHub Push
    ↓
GitHub Actions
    ↓
Install dependencies
    ↓
Astro Build
    ↓
Upload Pages Artifact
    ↓
Deploy GitHub Pages
```

不需要手动提交或上传 `dist/`。

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml
├── public/
├── src/
│   ├── content/
│   │   └── blog/
│   ├── layouts/
│   │   └── BaseLayout.astro
│   ├── pages/
│   │   ├── articles/
│   │   │   └── [...id].astro
│   │   ├── about.astro
│   │   ├── articles.astro
│   │   ├── index.astro
│   │   └── projects.astro
│   └── content.config.ts
├── astro.config.mjs
├── package.json
├── pnpm-lock.yaml
└── tsconfig.json
```

## Deployment

部署工作流位于：

```text
.github/workflows/deploy.yml
```

网站作为 GitHub User Site 部署在域名根路径：

```text
https://kailbug.github.io/
```

因此 Astro 配置包含 `site`，但不需要设置仓库子路径 `base`。
