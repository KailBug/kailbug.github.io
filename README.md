# kailbug.github.io

KailBug 的个人主页 <https://kailbug.github.io/>


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
