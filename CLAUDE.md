# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**重要规则：始终使用中文回复用户。**

## Project Overview

This is a Jekyll-based static site blog hosted on GitHub Pages at https://blog.bonza.cn. It's a personal technical blog with a custom forked theme originally by HyG (Gaohaoyang).

## Development Commands

| Command | Purpose |
|---------|---------|
| `jekyll s` / `jekyll serve` | Start local development server on port 4000 |
| `jekyll s --port 1234` | Serve on custom port |
| `bundle exec jekyll serve` | Serve with bundler (recommended) |
| `jekyll build` | Build the site to `_site/` |

On Windows, run `start.bat` to start the local server.

## Project Structure

- `_config.yml` - Main Jekyll configuration (site title, base URL, social links, comment system settings)
- `_posts/` - All published blog posts. Must follow naming convention `YYYY-MM-DD-title.md`
- `_layouts/` - HTML page layouts (default.html, post.html, page.html, nav.html, demo.html)
- `_includes/` - Reusable HTML components (header, footer, comments, reward, etc.)
- `_sass/` - SASS/SCSS partials for styling
- `_data/nav.yml` - Navigation menu structure with hierarchical categories
- `page/` - Special pages (archives, categories, tags, about, demo)
- `assets/images/` - Static images including reward QR codes

## Blog Post Format

All posts in `_posts/` must have this frontmatter format:

```yaml
---
layout: post
title: "Post Title"
date: "YYYY-MM-DD HH:MM"
category: category-name
tags: tag1 tag2 tag3
author: lework
---
* content
{:toc}

Description (summary)



Main content
```

**Important:**
- Post filename MUST follow `YYYY-MM-DD-title.md` format (Jekyll requirement)
- Four consecutive newlines define the excerpt separator for homepage summaries
- `* content` and `{:toc}` generate a table of contents
- The excerpt separator can be configured in `_config.yml` via `excerpt_separator`

## Configuration

The `_config.yml` file contains:
- Site metadata (title, tagline, base URL, email)
- Permalink format: `/:year/:month/:day/:title/`
- Comment system (Disqus, Gitalk, Utterances - only one should be enabled)
- Reward system (WeChat/Alipay QR codes)
- Analytics (Baidu Tongji, Google Analytics, Busuanzi)
- Social links configuration

## Theme Attribution

This blog uses a forked theme originally designed by HyG (Gaohaoyang): https://github.com/Gaohaoyang

## Deployment

- Branch: `gh-pages`
- Custom domain: `blog.bonza.cn` (defined in `CNAME`)
- Auto-deploys to GitHub Pages on push
