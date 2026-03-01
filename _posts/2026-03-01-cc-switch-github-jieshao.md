---
layout: post
title: "CC-Switch：Claude Code、Codex 和 Gemini CLI 全方位辅助工具"
date: "2026-03-01 15:15"
categories: tools
tags: [AI工具, Claude, Gemini, 开发工具, API管理, 跨平台]
author: lework
excerpt: 一个功能强大的跨平台桌面工具，用于管理 Claude Code、Codex 和 Gemini CLI 的 API 配置，支持 MCP 服务器、技能管理和提示词管理。
---

* content
{:toc}

CC-Switch 是一个功能全面的跨平台桌面应用程序，专为 Claude Code、Codex 和 Gemini CLI 用户设计。它提供了一个统一的界面来管理 API 配置、MCP 服务器、Claude 技能和系统提示词，大大简化了多 AI 工具的使用体验。




## 简介

![CC-Switch 界面](https://github.com/farion1231/cc-switch/raw/main/assets/screenshots/main-zh.png)




## 核心功能特性

### 🔧 供应商管理
- **一键切换**：在 Claude Code、Codex 和 Gemini 之间快速切换 API 配置
- **多云支持**：支持官方登录、AWS Bedrock 以及各种第三方 API 服务商
- **速度测试**：内置 API 端点延迟测试，帮助选择最优服务商

### 🌐 MCP 服务器管理
- **统一管理**：单一面板管理三个应用的 MCP 服务器
- **多传输协议**：支持 stdio、http 和 SSE 传输类型
- **智能同步**：自动同步配置到各应用的 live 文件

### 📚 Skills 管理（v3.7.0 新增）
- **自动发现**：从 GitHub 仓库自动扫描 Claude 技能
- **一键安装**：快速安装/卸载技能到 `~/.claude/skills/`
- **自定义仓库**：支持添加个人或团队的技能仓库

### 📝 Prompts 管理（v3.7.0 新增）
- **多预设支持**：创建无限数量的系统提示词预设
- **跨应用同步**：支持 Claude（CLAUDE.md）、Codex（AGENTS.md）和 Gemini（GEMINI.md）
- **实时预览**：内置 Markdown 编辑器，支持语法高亮和实时预览

### 🔄 高级功能
- **深度链接**：支持 `ccswitch://` 协议，一键导入配置
- **冲突检测**：自动检测跨应用配置冲突并提供解决方案
- **云同步**：支持通过 Dropbox、OneDrive 等实现跨设备同步
- **系统托盘**：快速切换供应商，支持托盘菜单操作

## 技术架构

### 前端技术栈
- **React 18** + TypeScript
- **Vite** 构建工具
- **TailwindCSS** 样式框架
- **TanStack Query** 状态管理
- **shadcn/ui** 组件库

### 后端技术栈
- **Tauri 2.8** 跨平台框架
- **Rust** 系统编程语言
- **SQLite** 数据存储
- **Tokio** 异步运行时

### 架构设计原则
```
┌─────────────────────────────────────────────────────────────┐
│                    前端 (React + TS)                         │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │ Components  │  │    Hooks     │  │  TanStack Query  │    │
│  │   （UI）     │──│ （业务逻辑）   │──│   （缓存/同步）    │    │
│  └─────────────┘  └──────────────┘  └──────────────────┘    │
└────────────────────────┬────────────────────────────────────┘
                         │ Tauri IPC
┌────────────────────────▼────────────────────────────────────┐
│                  后端 (Tauri + Rust)                         │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │  Commands   │  │   Services   │  │  Models/Config   │    │
│  │ （API 层）   │──│  （业务层）    │──│    （数据）       │    │
│  └─────────────┘  └──────────────┘  └──────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## 安装指南

### 系统要求
- **Windows**: Windows 10 及以上
- **macOS**: macOS 10.15 (Catalina) 及以上  
- **Linux**: Ubuntu 22.04+ / Debian 11+ / Fedora 34+ 等

### 安装方式

#### macOS 用户（推荐使用 Homebrew）
```bash
brew tap farion1231/ccswitch
brew install --cask cc-switch

# 更新
brew upgrade --cask cc-switch
```

#### Windows 用户
从 [Releases](https://github.com/farion1231/cc-switch/releases) 页面下载：
- `CC-Switch-v{版本号}-Windows.msi`（安装版）
- `CC-Switch-v{版本号}-Windows-Portable.zip`（绿色版）

#### Linux 用户
根据发行版选择对应的安装包：
- `CC-Switch-v{版本号}-Linux.deb`（Debian/Ubuntu）
- `CC-Switch-v{版本号}-Linux.rpm`（Fedora/RHEL）
- `CC-Switch-v{版本号}-Linux.AppImage`（通用）

## 快速开始

### 基本使用流程

1. **添加供应商**
   - 点击"添加供应商"按钮
   - 选择预设配置或创建自定义配置
   - 输入 API 密钥和相关设置

2. **切换供应商**
   - **主界面**：选择供应商 → 点击"启用"
   - **系统托盘**：直接点击供应商名称（立即生效）
   - 重启终端或对应客户端以应用更改

3. **MCP 服务器管理**
   - 点击右上角"MCP"按钮
   - 使用内置模板或添加自定义服务器
   - 启用/禁用服务器并同步到 live 配置

### 高级配置

#### 云同步设置
1. 前往设置 → "自定义配置目录"
2. 选择云同步文件夹（Dropbox、OneDrive、iCloud 等）
3. 重启应用以应用更改
4. 在其他设备上重复操作以启用跨设备同步

#### 环境变量冲突检测
- 自动检测跨应用配置冲突
- 提供可视化冲突指示器
- 给出解决方案建议
- 更改前自动备份

## 版本亮点

### v3.8.0 重大更新（2025-11-28）

**持久化架构升级 & 全新用户界面**

- **SQLite + JSON 双层架构**：为未来云同步功能奠定基础
- **全新用户界面**：完全重新设计的界面布局和组件样式
- **日语支持**：新增日语界面支持
- **开机自启**：一键开启/关闭自启动功能

### v3.7.0 核心功能

**六大核心功能，18,000+ 行新增代码**

- **Gemini CLI 集成**：第三个支持的 AI CLI
- **Claude Skills 管理系统**：从 GitHub 仓库自动扫描技能
- **Prompts 管理系统**：多预设系统提示词管理
- **MCP v3.7.0 统一架构**：单一面板管理三个应用的 MCP 服务器
- **深度链接协议**：`ccswitch://` 协议注册
- **环境变量冲突检测**：自动检测跨应用配置冲突

## 开发信息

### 环境要求
- Node.js 18+
- pnpm 8+
- Rust 1.85+
- Tauri CLI 2.8+

### 开发命令
```bash
# 安装依赖
pnpm install

# 开发模式（热重载）
pnpm dev

# 类型检查
pnpm typecheck

# 代码格式化
pnpm format

# 运行测试
pnpm test:unit

# 构建应用
pnpm build
```

## 项目特色

### 🎯 智能化程度高
- 自动内容分析和处理
- 智能分类和标签生成
- 质量自动评估和优化

### 📋 规范遵循严格
- 完整的 Jekyll 格式支持
- 多层验证机制
- 详细的规范文档

### 🚀 自动化程度高
- 一键式处理流程
- 自动部署和监控
- 错误自动恢复

### 🔧 可扩展性强
- 模块化设计架构
- 易于定制和扩展
- 丰富的配置选项

## 总结

CC-Switch 是一个功能强大、设计精良的 AI 工具管理平台，它解决了多 AI 工具使用过程中的配置管理难题。无论是个人开发者还是团队协作，都能从中受益，提高工作效率。

通过统一的界面管理多个 AI 工具的配置，支持 MCP 服务器、技能管理和提示词管理，CC-Switch 真正实现了"一站式"AI 工具管理体验。

---

**项目地址**：[https://github.com/farion1231/cc-switch](https://github.com/farion1231/cc-switch)

**许可证**：MIT © Jason Young

**下载地址**：[GitHub Releases](https://github.com/farion1231/cc-switch/releases)