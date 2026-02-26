---
layout: post
title:  "GitHub Store：跨平台 GitHub 版本应用商店"
date:   2026-02-26 16:55:00 +0800
categories: 开源项目
tags: github android kotlin 跨平台 应用商店
author: 来财
---

GitHub Store 是一个基于 GitHub Releases 的跨平台应用商店，旨在简化开源软件的发现和安装过程。该项目使用 Kotlin Multiplatform 和 Compose Multiplatform 构建，支持 Android 和桌面平台，能够自动检测可安装的二进制文件，提供一键安装、更新追踪等功能，以类似应用商店的界面展示 GitHub 仓库信息。本文将详细介绍这个开源项目的主要特性、工作原理和使用场景。




* content
{:toc}

## 项目概览

GitHub Store 是由高中生开发者 rainxchzed 创建的开源项目，目前拥有 48,000+ 活跃用户和 5,500+ GitHub Star。项目完全免费，无广告、无跟踪、无付费功能，为开发者社区提供了一个纯净的 GitHub 软件发现平台。

### 核心特性

1. **跨平台支持** - 使用 Kotlin Multiplatform 构建，支持 Android、Windows、macOS、Linux
2. **智能发现** - 首页展示"热门"、"最近更新"、"新项目"等板块，基于时间过滤
3. **一键安装** - 自动检测最新版本的二进制文件，支持 APK、EXE、DMG、AppImage、DEB、RPM
4. **应用追踪** - 在 Android 平台本地追踪已安装的应用，高亮显示更新提示
5. **统一体验** - 跨平台一致的用户界面和交互逻辑

## 主要功能

### 智能应用发现

- **首页分区展示** - "热门"（Trending）、"最近更新"（Recently Updated）、"新项目"（New）三个主要板块
- **时间过滤** - 支持基于时间的动态筛选，展示最新动态
- **有效资源过滤** - 只显示包含有效可安装资源的仓库
- **平台感知评分** - 根据当前平台对 topic 进行评分，优先展示相关的应用

### 最新版本安装

- **自动获取最新版本** - 调用 GitHub API 的 `/releases/latest` 端点
- **资源筛选** - 只显示最新版本的安装包资源
- **一键安装** - 提供"安装最新版本"快速操作
- **完整列表** - 可展开查看该版本的所有安装选项

### 详细信息页面

- **应用基本信息** - 名称、版本、"安装最新版本"按钮
- **仓库指标** - Star 数、Fork 数、Open Issues 数
- **应用说明** - 渲染 README 内容作为"关于此应用"
- **版本说明** - 最新版本的发布说明，支持 Markdown 格式
- **安装列表** - 显示所有安装选项，包含平台标签和文件大小

### 跨平台用户体验

**Android 平台：**
- 使用包安装器打开 APK 下载
- 在本地数据库中跟踪安装的应用
- 在专门的"已安装应用"界面展示，含更新提示

**桌面平台（Windows/macOS/Linux）：**
- 下载安装包到用户下载文件夹
- 使用系统默认处理器打开
- 不使用隐藏的临时目录

## 工作原理

### 1. 应用发现流程

GitHub Store 通过以下步骤发现应用：

1. **搜索** - 使用 GitHub 的 `/search/repositories` 端点执行平台感知查询
2. **评分** - 基于 topic、语言和描述进行简单评分
3. **过滤** - 排除已归档的仓库和信号不足的仓库

### 2. 发布版本与资源检查

对候选仓库执行以下检查：

1. **获取最新版本** - 调用 `/repos/{owner}/{repo}/releases/latest`
2. **资源扫描** - 检查 `assets` 数组中的平台特定文件扩展名
3. **排除无效仓库** - 如果没有找到合适的资源文件，从结果中排除

### 3. 详细信息展示

综合展示多方面信息：

- **仓库信息**：名称、所有者、描述、Stars、Forks、Issues
- **最新版本**：标签、发布日期、正文（changelog）、资源
- **README**：从默认分支加载并渲染为"关于此应用"

### 4. 安装流程

当用户点击"安装最新版本"时：

1. **平台匹配** - 为当前平台选择最佳匹配的资源
2. **流式下载** - 流式传输下载内容
3. **系统安装** - 委托给操作系统的安装器
   - Android：APK 安装器
   - 桌面：系统默认处理器
4. **安装追踪** - 在 Android 平台记录安装到本地数据库

## 如何让你的应用出现在 GitHub Store

GitHub Store 不使用私有索引或人工策划规则，只要你的项目符合以下条件，就可以自动出现：

### 条件要求

1. **GitHub 上的公开仓库**
   - 可见性必须为 `public`

2. **至少一个已发布的版本**
   - 通过 GitHub Releases 创建（不仅仅是标签）
   - 最新版本不能是草稿（draft）或预发布版本（prerelease）

3. **最新版本中包含可安装资源**
   - 最新版本必须包含至少一个具有受支持扩展名的资源文件：
     - Android: `.apk`
     - Windows: `.exe`, `.msi`
     - macOS: `.dmg`, `.pkg`
     - Linux: `.deb`, `.rpm`, `.AppImage`
   - GitHub Store 会忽略 GitHub 自动生成的源代码归档文件（`Source code (zip)` / `Source code (tar.gz)`）

4. **可通过搜索/topic 发现**
   - 仓库通过公共 GitHub Search API 获取
   - Topic、语言和描述有助于排名：
     - Android 应用：topics 如 `android`、`mobile`、`apk`
     - 桌面应用：topics 如 `desktop`、`windows`、`linux`、`macos`、`compose-desktop`、`electron`
   - 拥有至少一些 Star 会增加在热门/更新/新版块出现的可能性

如果你的仓库满足这些条件，GitHub Store 就可以通过搜索找到它并自动展示，无需手动提交。

## 使用优势

### 相比传统方式的优势

1. **无需在 GitHub Releases 中搜寻**
   - 只显示实际为你的平台发布二进制文件的仓库

2. **了解你安装了什么**
   - 追踪通过 GitHub Store 安装的应用（Android）
   - 高亮显示新版本可用时，方便更新

3. **始终安装最新版本**
   - 安装保证来自最新发布的版本
   - 你看到的版本日志正是你安装的内容

4. **跨平台统一体验**
   - Android 和桌面具有相同的 UI 和逻辑
   - 具有平台原生的安装行为

5. **开源且可扩展**
   - 使用 KMP 编写，网络、领域逻辑和 UI 清晰分离
   - 易于 fork、扩展或适配

## 技术架构

### 技术栈

- **Kotlin Multiplatform** - 跨平台代码共享
- **Compose Multiplatform** - 声明式 UI 框架
- **GitHub API** - 仓库搜索和版本信息获取
- **Material You** - Material Design 3 设计语言

### 平台支持

- **Android API 24+** - Android 7.0 及以上版本
- **Windows 10+**
- **macOS 11+**
- **Linux（主流发行版）**

### 签名与安全

所有官方 GitHub Store 版本都使用以下证书指纹签名：

SHA-256:
`B7:F2:8E:19:8E:48:C1:93:B0:38:C6:5D:92:DD:F7:BC:07:7B:0D:B5:9E:BC:9B:25:0A:6D:AC:48:C1:18:03:CA`

**macOS 用户注意：**
你可能看到警告提示 Apple 无法验证 GitHub Store。这是因为应用在 App Store 外分发且尚未进行公证。请通过以下步骤允许：系统设置 → 隐私与安全性 → 仍要打开。

## GitHub OAuth 配置

如需开发 GitHub Store，需要配置 GitHub OAuth：

### 快速配置步骤

1. **创建 GitHub OAuth App**
   - GitHub → 设置 → 开发者设置 → OAuth Apps → 新建 OAuth App
   - Application name: 任意（如 *GitHub Store Dev*）
   - Homepage URL: `https://github.com/username/repo_name`
   - Authorization callback URL: `githubstore://callback`

2. **复制 Client ID**
   - 只需要 Client ID（不需要 Client Secret）

3. **添加到项目**
   - 在项目根目录的 `local.properties` 文件中添加：
   ```properties
   GITHUB_CLIENT_ID=YOUR_CLIENT_ID_HERE
   ```

4. **同步并运行**
   - 同步项目并运行应用，现在可以使用 GitHub 登录

## 社区与支持

### 下载渠道

- **GitHub Releases**: [rainxchzed/Github-Store](https://github.com/rainxchzed/Github-Store/releases)
- **F-Droid**: [zed.rainxch.githubstore](https://f-droid.org/packages/zed.rainxch.githubstore/)
- **Obtainium**: 支持通过 Obtainium 管理更新

### 社区资源

- **官网**: [https://github-store.org](https://github-store.org)
- **Discord**: [加入社区](https://discord.gg/x9Cvh2Z9qS)
- **Wiki**: [GitHub Store Wiki](https://github.com/rainxchzed/Github-Store/wiki)
- **隐私政策**: [github-store.org/privacy-policy](https://github-store.org/privacy-policy/)

### 媒体报道

- **HowToMen**: [2026 年最佳 20 个 Android 应用](https://www.youtube.com/watch?v=7favc9MDedQ)
- **HowToMen**: [比 Google Play 商店更好的 12 个应用商店](https://www.youtube.com/watch?v=VR-MEwPDw4k)
- **HelloGitHub**: [精选项目](https://hellogithub.com/en/repository/rainxchzed/Github-Store)

## 项目支持

GitHub Store 已经拥有 **48,000+ 活跃用户**和 **5,500+ GitHub Star**，并且是 **100% 免费**的，无广告、无跟踪、无付费功能。

项目作者在完成高中学业期间独立开发和维护这个项目。你的支持（即使只是 $3）可以帮助：

✅ 保持应用无 bug —— 快速响应问题并发布修复  
✅ 添加社区需要的功能 —— 实现用户真正需要的内容  
✅ 维护基础设施 —— 服务器、API 和部署成本

### 支持方式

- Buy Me a Coffee: [https://www.buymeacoffee.com/rainxchzed](https://www.buymeacoffee.com/rainxchzed)
- GitHub Sponsors: [https://github.com/sponsors/rainxchzed](https://github.com/sponsors/rainxchzed)

**无法赞助？没关系！** 你仍然可以通过以下方式帮助：
- ⭐ **给这个仓库点星** —— 帮助其他人发现 GitHub Store
- 🐛 **报告 bug** —— 让应用对每个人都更好
- 📢 **分享给朋友** —— 向其他开发者传播这个工具
- 💬 **加入 [Discord](https://discord.gg/x9Cvh2Z9qS)** —— 你的反馈决定了路线图

## 免责声明

GitHub Store 只帮助你发现和下载第三方开发者在 GitHub 上已经发布的版本资源。那些下载的内容、安全性和行为完全由各自的作者和分发者负责，而非本项目。

通过使用 GithubStore，你理解并同意你以自己的风险安装和运行任何下载的软件。本项目不审查、验证或保证任何安装程序是安全的、无恶意软件的或适合任何特定用途。

## 许可证

GitHub Store 将在 **Apache License, Version 2.0** 下发布。

```
Copyright 2025 rainxchzed

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this project except in compliance with the License.
You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## 总结

GitHub Store 是一个创新的开源项目，通过现代化的技术栈和精心设计的用户体验，为开发者社区提供了一个便捷的 GitHub 软件发现和安装平台。项目展现了跨平台开发、自动化软件分发、社区驱动的应用生态等多方面的技术亮点。

由高中生开发者独立完成并获得 48,000+ 活跃用户和 5,500+ GitHub Star 的成绩，这个项目充分展示了开源社区的活力和年轻开发者的创造力。无论你是寻找开源软件的应用用户，还是想要了解跨平台应用开发的开发者，GitHub Store 都是一个值得关注的项目。

## 相关资源

- **GitHub 仓库**: https://github.com/rainxchzed/Github-Store
- **官方网站**: https://github-store.org
- **完整文档**: https://github.com/rainxchzed/Github-Store/wiki
- **许可证**: Apache License 2.0
- **技术栈**: Kotlin Multiplatform, Compose Multiplatform, GitHub API

**标签**: `github` `kotlin` `multimodul` `android` `desktop` `app-store` `open-source`
