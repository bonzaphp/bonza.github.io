---
layout: post
title:  "CodePilot：Claude Code 的原生桌面 GUI，可视化编程助手"
date:   2026-02-12 03:39:32 +0800
categories: github
tags: github claude codepilot electron next.js 桌面应用 ai编程
author: 来财
---
* content
{:toc}



CodePilot 是一个专为 Claude Code 设计的原生桌面图形用户界面，让你可以通过精美的可视化界面进行聊天、编码和项目管理，而不是通过终端。这是一个基于 Electron + Next.js 构建的现代化桌面应用。

## 项目简介

**项目名称**: CodePilot  
**项目地址**: [https://github.com/op7418/CodePilot](https://github.com/op7418/CodePilot)  
**项目描述**: A native desktop GUI for Claude Code — chat, code, and manage projects visually. Built with Electron + Next.js.  
**开发语言**: TypeScript/JavaScript  
**开源协议**: MIT  

CodePilot 为 Claude Code 提供了一个直观的桌面环境，让开发者能够更高效地与 AI 助手协作，通过图形化界面管理对话、文件和项目，大大提升了开发体验。

## 核心特性

### 🚀 对话式编程
- **实时流式响应**：实时接收 Claude 的响应，支持完整的 Markdown 渲染
- **语法高亮**：代码块具有语法高亮功能
- **工具调用可视化**：直观显示 AI 工具调用过程

### 💾 会话管理
- **会话持久化**：创建、重命名、归档和恢复聊天会话
- **本地存储**：对话内容存储在本地 SQLite 数据库中
- **重启恢复**：应用重启后所有对话都会保留

### 📁 项目感知上下文
- **工作目录选择**：为每个会话选择工作目录
- **实时文件树**：右侧面板显示实时文件树和文件预览
- **上下文感知**：随时了解 Claude 正在查看的文件内容

### 🎛️ 可调整面板
- **拖拽调整**：拖动聊天列表和右侧面板边缘调整宽度
- **尺寸保存**：你的偏好设置会在会话间保存

### 📎 文件和图片附件
- **直接附件**：在聊天输入中直接附加文件和图片
- **多模态分析**：图片作为多模态视觉内容发送给 Claude 分析

### 🔐 权限控制
- **精细化权限**：批准、拒绝或自动允许工具使用
- **按动作控制**：基于每个动作的权限控制
- **权限模式**：选择不同的权限模式以匹配你的舒适度

### 🔄 多种交互模式
- **Code 模式**：编程模式
- **Plan 模式**：规划模式  
- **Ask 模式**：问答模式
- **模式切换**：控制 Claude 在每个会话中的行为方式

### 🤖 模型选择器
- **模型切换**：在对话中切换 Claude 模型（Opus、Sonnet、Haiku）
- **实时切换**：无需重启会话即可切换模型

### 🔌 MCP 服务器管理
- **直接管理**：从扩展页面直接添加、配置和删除 Model Context Protocol 服务器
- **多传输支持**：支持 stdio、sse 和 http 传输类型

### 🛠️ 自定义技能
- **可重用技能**：定义基于提示的可重用技能（全局或项目特定）
- **斜杠命令**：在聊天中作为斜杠命令调用技能

### ⚙️ 设置编辑器
- **可视化编辑**：可视化编辑器用于 ~/.claude/settings.json
- **JSON 编辑**：支持 JSON 格式编辑
- **环境变量**：包括权限和环境变量配置

### 📊 Token 使用跟踪
- **实时统计**：每次助手响应后显示输入/输出 token 数量
- **成本估算**：显示预估的使用成本

### 🔄 自动更新检查
- **定期检查**：应用定期检查新版本
- **更新通知**：有更新时通知用户

### 🌙 主题支持
- **深色/浅色主题**：导航栏中一键切换主题
- **主题持久化**：主题选择会保存

### ⚡ 斜杠命令
- **内置命令**：/help、/clear、/cost、/compact、/doctor、/review 等
- **快速操作**：通过斜杠命令快速执行常用操作

### 📦 Electron 打包
- **原生应用**：作为原生桌面应用分发
- **隐藏标题栏**：隐藏标题栏的现代界面
- **集成服务器**：捆绑的 Next.js 服务器
- **优雅关闭**：优雅关闭机制
- **自动端口分配**：自动分配可用端口

## 技术栈

### 🏗️ 核心技术

| 层级 | 技术 |
|------|------|
| **框架** | [Next.js 16](https://nextjs.org/) (App Router) |
| **桌面外壳** | [Electron 40](https://www.electronjs.org/) |
| **UI 组件** | [Radix UI](https://www.radix-ui.com/) + [shadcn/ui](https://ui.shadcn.com/) |
| **样式** | [Tailwind CSS 4](https://tailwindcss.com/) |
| **动画** | [Motion](https://motion.dev/) (Framer Motion) |
| **AI 集成** | [Claude Agent SDK](https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk) |
| **数据库** | [better-sqlite3](https://github.com/WiseLibs/better-sqlite3) (嵌入式，每用户) |
| **Markdown** | react-markdown + remark-gfm + rehype-raw + [Shiki](https://shiki.style/) |
| **流式传输** | [Vercel AI SDK](https://sdk.vercel.ai/) helpers + Server-Sent Events |
| **图标** | [Hugeicons](https://hugeicons.com/) + [Lucide](https://lucide.dev/) |
| **测试** | [Playwright](https://playwright.dev/) |
| **CI/CD** | [GitHub Actions](https://github.com/features/actions) (自动构建 + 标签发布) |
| **构建/打包** | electron-builder + esbuild |

### 🏛️ 项目结构

```
codepilot/
├── .github/workflows/     # CI/CD: 多平台构建 & 自动发布
├── electron/              # Electron 主进程 & 预加载
│   ├── main.ts           # 窗口创建、嵌入式服务器生命周期
│   └── preload.ts        # 上下文桥接
├── src/
│   ├── app/              # Next.js App Router 页面 & API 路由
│   │   ├── chat/         # 新建聊天页面 & [id] 会话页面
│   │   ├── extensions/   # 技能 + MCP 服务器管理
│   │   ├── settings/     # 设置编辑器
│   │   └── api/          # REST + SSE 端点
│   │       ├── chat/     # 会话、消息、流式传输、权限
│   │       ├── files/    # 文件树 & 预览
│   │       ├── plugins/  # 插件 & MCP CRUD
│   │       ├── settings/ # 设置读写
│   │       ├── skills/   # 技能 CRUD
│   │       └── tasks/    # 任务跟踪
│   ├── components/       # React 组件
│   │   ├── ai-elements/  # 消息气泡、代码块、工具调用等
│   │   ├── chat/         # ChatView、MessageList、MessageInput、流式传输
│   │   ├── layout/       # AppShell、NavRail、ResizeHandle、RightPanel
│   │   ├── plugins/      # MCP 服务器列表 & 编辑器
│   │   ├── project/      # FileTree、FilePreview、TaskList
│   │   ├── skills/       # SkillsManager、SkillEditor
│   │   └── ui/           # 基于 Radix 的基础组件
│   ├── hooks/            # 自定义 React hooks
│   ├── lib/              # 核心逻辑
│   │   ├── claude-client.ts    # Agent SDK 流式包装器
│   │   ├── db.ts              # SQLite 模式、迁移、CRUD
│   │   ├── files.ts           # 文件系统助手
│   │   ├── permission-registry.ts # 权限请求/响应桥接
│   │   └── utils.ts           # 共享工具
│   └── types/            # TypeScript 接口 & API 契约
├── electron-builder.yml  # 打包配置
├── package.json
└── tsconfig.json
```

## 系统要求

### 📋 前置条件

**重要提示**：CodePilot 底层调用 Claude Code Agent SDK。确保 claude 在你的 PATH 中可用，并且你已通过身份验证（claude login），然后再启动应用。

| 要求 | 最低版本 |
|------|----------|
| **Node.js** | 18+ |
| **Claude Code CLI** | 已安装并已验证身份（claude --version 应该可用） |
| **npm** | 9+ (随 Node 18 一起提供) |

## 下载和安装

### 📥 下载地址

预构建的发布版本可在 [Releases](https://github.com/op7418/CodePilot/releases) 页面获得。发布版本通过 GitHub Actions 为所有平台自动构建。

### 🖥️ 支持的平台

- **macOS** -- arm64 (Apple Silicon) 和 x64 (Intel) 以 .dmg 形式分发
- **Windows** -- NSIS 安装程序 (.exe) 打包 x64 + arm64
- **Linux** -- x64 和 arm64 以 .AppImage、.deb 和 .rpm 形式分发

## 快速开始

### 🚀 开发环境设置

```bash
# 克隆仓库
git clone https://github.com/op7418/CodePilot.git
cd CodePilot

# 安装依赖
npm install

# 开发模式启动（浏览器）
npm run dev

# -- 或在开发模式下启动完整的 Electron 应用 --
npm run electron:dev
```

然后打开 [http://localhost:3000](http://localhost:3000)（浏览器模式）或等待 Electron 窗口出现。

### 🔧 开发命令

```bash
# 仅运行 Next.js 开发服务器（在浏览器中打开）
npm run dev

# 在开发模式下运行完整的 Electron 应用
# (启动 Next.js + 等待它，然后打开 Electron)
npm run electron:dev

# 生产构建（Next.js 静态导出）
npm run build

# 构建 Electron 分发版 + Next.js
npm run electron:build

# 为特定平台打包
npm run electron:pack:mac    # macOS DMG (arm64 + x64)
npm run electron:pack:win    # Windows NSIS 安装程序
npm run electron:pack:linux  # Linux AppImage, deb, rpm
```

## 安装故障排除

### ⚠️ 安全警告

CodePilot 尚未代码签名，因此首次打开时操作系统会显示安全警告。

#### macOS

你会看到一个对话框，提示"Apple cannot check it for malicious software"。

**选项 1 -- 右键打开**
- 在 Finder 中右键（或 Control-点击）CodePilot.app
- 从上下文菜单中选择打开
- 在确认对话框中点击打开

**选项 2 -- 系统设置**
- 打开系统设置 > 隐私与安全
- 向下滚动到安全部分
- 你会看到关于 CodePilot 被阻止的消息。点击仍要打开
- 如果出现提示，请进行身份验证，然后启动应用

**选项 3 -- 终端命令**
```bash
xattr -cr /Applications/CodePilot.app
```
这会移除隔离属性，因此 macOS 不再阻止该应用。

#### Windows

Windows SmartScreen 会阻止安装程序或可执行文件。

**选项 1 -- 仍要运行**
- 在 SmartScreen 对话框中，单击更多信息
- 单击仍要运行

**选项 2 -- 禁用应用安装控制**
- 打开设置 > 应用 > 高级应用设置
- 切换应用安装控制（或"选择从何处获取应用"）以允许来自任何位置的应用

## CI/CD 和自动化

### 🔄 自动化构建

项目使用 GitHub Actions 进行自动化构建。推送 v* 标签会触发完整的多平台构建并自动创建包含所有工件的 GitHub Release：

```bash
git tag v0.8.1
git push origin v0.8.1
# CI 构建 Windows + macOS + Linux，然后发布版本
```

你也可以从 Actions 选项卡手动触发单个平台的构建。

### 📝 开发说明

- Electron 主进程（electron/main.ts）分叉 Next.js 独立服务器并通过 127.0.0.1 上的随机空闲端口连接到它。
- 聊天数据存储在 ~/.codepilot/codepilot.db 中（或在开发模式中存储在 ./data/codepilot.db）。
- 应用对 SQLite 使用 WAL 模式，因此并发读取速度很快。

## 贡献指南

### 🤝 如何贡献

欢迎贡献。要开始：

1. **Fork 仓库**并创建功能分支
2. 使用 `npm install` 安装依赖
3. 运行 `npm run electron:dev` 在本地测试你的更改
4. 在打开 pull request 之前确保 `npm run lint` 通过
5. 针对主分支打开 PR，清楚说明更改内容和原因

请保持 PR 专注——每个 pull request 一个功能或修复。

## 许可证

本项目采用 **MIT** 许可证。

## 总结

CodePilot 是一个功能强大、设计精美的 Claude Code 桌面客户端，它将命令行体验提升到了现代化的图形界面。通过直观的可视化操作、丰富的功能和优秀的用户体验，CodePilot 让 AI 辅助编程变得更加高效和愉快。

### 🎯 核心优势

1. **🎨 现代化界面**：基于 Electron + Next.js 的现代化桌面应用
2. **🔄 实时协作**：与 Claude 的实时流式对话体验
3. **📁 项目管理**：直观的文件管理和项目上下文
4. **🔌 扩展性**：支持 MCP 服务器和自定义技能
5. **💾 数据持久化**：本地 SQLite 数据库存储
6. **🌐 多平台支持**：Windows、macOS、Linux 全平台支持

### 🚀 适用场景

- **日常开发**：AI 辅助编程和代码生成
- **项目管理**：可视化的项目文件管理
- **学习研究**：AI 辅助的技术学习和研究
- **团队协作**：共享的 AI 编程助手

如果你正在寻找一个功能丰富、界面美观的 Claude Code 桌面客户端，CodePilot 绝对是最佳选择！

---

*本文基于 CodePilot 项目的官方文档编写，如有更新请以官方文档为准。*
