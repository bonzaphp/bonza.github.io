# 博客发布规范

## 文章格式要求

### 首页折叠显示
**重要**：所有发布的文章必须在首页显示为折叠形式，而不是展开。

**实现方法**：在文章的 front matter 和正文内容之间添加**四个连续的换行符**。

#### 正确格式：
```markdown
---
layout: post
title:  "文章标题"
date:   2026-02-09 02:40:00 +0800
categories: docker
tags: docker example
author: 来财
---


* content
{:toc}

文章正文内容...
```

#### 错误格式（会导致首页展开）：
```markdown
---
layout: post
title:  "文章标题"
date:   2026-02-09 02:40:00 +0800
categories: docker
tags: docker example
author: 来财
---

* content
{:toc}

文章正文内容...
```

### 原因说明
- Jekyll 配置文件中设置了 `excerpt_separator: "\n\n\n\n"`
- 只有当遇到四个连续换行符时，Jekyll 才会将其作为摘要分隔符
- 摘要之前的内容会在首页显示，之后的内容会折叠隐藏

## 文件命名规范
- 格式：`YYYY-MM-DD-文章标题.md`
- 示例：`2026-02-09-docker-compose-gotify-deployment.md`

## 类目设置
- 在 front matter 中设置 `categories` 字段
- 示例：`categories: docker`

## 提交信息规范
- 格式：`添加 [文章标题] - YYYY年M月D日`
- 示例：`添加 Docker Compose 部署 Gotify 文章 - 2026年2月9日`

## 发布流程
1. 创建文章文件（确保添加四个换行符）
2. 添加到 git：`git add _posts/YYYY-MM-DD-文章标题.md`
3. 提交：`git commit -m "提交信息"`
4. 推送：`git push origin gh-pages`

---
*创建时间: 2026年2月9日*
*记录者: 来财*