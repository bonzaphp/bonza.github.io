---
layout: post
title:  "Memoh：多成员、结构化长期记忆的容器化 AI 智能体系统"
date:   2026-02-12 03:17:50 +0800
categories: ai-agent
tags: ai-agent memoh 容器化 多机器人 长期记忆 golang
author: 来财
---
* content
{:toc}



Memoh 是一个强大的 AI 智能体系统平台，用户可以创建自己的 AI 机器人，并通过 Telegram、Discord、Lark(飞书)等平台与它们聊天。每个机器人都有独立的容器和内存系统，允许它们编辑文件、执行命令并自我构建——类似于 OpenClaw，Memoh 为多机器人管理提供了更安全、更灵活、更可扩展的解决方案。

## 项目简介

**项目名称**: Memoh  
**项目地址**: [https://github.com/memohai/Memoh](https://github.com/memohai/Memoh)  
**项目描述**: Multi-Member, Structured Long-Memory, Containerized AI Agent System ✨  
**开发语言**: Golang  
**开源协议**: AGPLv3  

Memoh 是一个用 Golang 构建的多机器人智能体服务，提供机器人、频道、MCP 和技能的完整图形化配置。我们使用 Containerd 为每个机器人提供容器级别的隔离，并深受 OpenClaw 的 Agent 设计启发。

## 为什么选择 Memoh？

### 现有方案的局限性

OpenClaw、Clawdbot 和 Moltbot 虽然令人印象深刻，但它们存在明显的缺点：稳定性问题、安全顾虑、配置复杂以及高昂的 token 成本。如果您正在寻找一个稳定、安全的机器人 SaaS 解决方案，可以考虑我们的开源 Memoh。

### Memoh 的优势

#### 🚀 核心特性
- **多机器人管理**：创建多个机器人；人类和机器人，或机器人之间，可以进行私人聊天、群聊或协作。
- **容器化隔离**：每个机器人在自己的独立容器中运行。机器人可以在容器内自由执行命令、编辑文件和访问网络——就像拥有自己的计算机一样。
- **记忆工程**：每次聊天都存储在数据库中，默认加载最近 24 小时的上下文。每次对话轮次都存储为记忆，机器人可以通过语义搜索检索。
- **多平台支持**：支持 Telegram、Lark (飞书) 等多个平台。
- **简单易用**：通过图形界面配置机器人和 Provider、Model、Memory、Channel、MCP 和 Skills 设置——无需编码即可设置自己的 AI 机器人。
- **定时任务**：使用 cron 表达式调度任务，在指定时间运行命令。

## 技术架构

### 🏗️ 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Telegram      │    │    Discord      │    │   Lark/Feishu   │
│                 │    │                 │    │                 │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────┴─────────────┐
                    │      Memoh Core          │
                    │    (Golang Backend)      │
                    └─────────────┬─────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │    Containerd Runtime    │
                    │  (Container Isolation)   │
                    └─────────────┬─────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │     Bot Containers       │
                    │  ┌─────┐ ┌─────┐ ┌─────┐ │
                    │  │Bot1 │ │Bot2 │ │Bot3 │ │
                    │  └─────┘ └─────┘ └─────┘ │
                    └───────────────────────────┘
```

### 💾 记忆系统

Memoh 机器人具有深度工程化的记忆层，灵感来自 Mem0。通过存储每次对话轮次的知识，它能够实现更精确的记忆检索。

#### 记忆特性
- **持久化存储**：所有对话都存储在数据库中
- **上下文加载**：默认加载最近 24 小时的上下文
- **语义搜索**：通过语义搜索检索记忆
- **多用户识别**：能够区分和记住来自多个人类和机器人的请求
- **群聊协作**：在任何群聊中无缝工作

## 核心功能详解

### 🤖 多机器人管理

Memoh 支持创建和管理多个 AI 机器人，每个机器人都有：

- **独立身份**：独立的配置和设置
- **容器隔离**：完全隔离的运行环境
- **专属记忆**：独立的记忆系统
- **平台集成**：可连接到不同的聊天平台

### 🐳 容器化架构

每个机器人运行在独立的容器中，提供：

```bash
# 容器隔离示例
docker run -d \
  --name bot-1 \
  --memory=512m \
  --cpus=0.5 \
  -v bot-1-data:/app/data \
  memoh/bot:latest
```

**容器优势**：
- **安全性**：完全隔离的执行环境
- **稳定性**：单个机器人崩溃不影响其他机器人
- **资源控制**：精确的 CPU 和内存限制
- **文件系统**：独立的文件系统访问

### 🧠 记忆工程

Memoh 的记忆系统包括：

#### 记忆存储
```yaml
memory:
  type: vector_database
  retention: 30d
  context_window: 24h
  embedding_model: text-embedding-3-small
```

#### 记忆检索
```python
# 语义搜索示例
memories = bot.search_memories(
    query="用户之前提到的项目",
    limit=5,
    threshold=0.7
)
```

### 📱 多平台支持

#### 支持的平台
- **Telegram**：完整的 Bot API 支持
- **Discord**：Slash Commands 和 Webhooks
- **Lark/Feishu**：企业级协作平台
- **更多平台**：持续扩展中

#### 平台集成示例
```yaml
# Telegram 配置
telegram:
  bot_token: "your_bot_token"
  webhook_url: "https://your-domain.com/webhook/telegram"
  
# Discord 配置
discord:
  bot_token: "your_discord_token"
  guild_ids: ["your_guild_id"]
```

## 快速开始

### 🐳 Docker 部署（推荐）

最快的部署方式：

```bash
# 克隆项目
git clone https://github.com/memohai/Memoh.git
cd Memoh

# 一键部署
./deploy.sh

# 访问管理界面
open http://localhost
```

### 📋 部署要求

#### 系统要求
- **操作系统**: Linux, macOS, Windows
- **Docker**: 20.10+ 
- **Docker Compose**: 2.0+
- **内存**: 至少 2GB 可用内存
- **磁盘**: 至少 10GB 可用空间

#### 网络要求
- **出站网络**: 访问 AI API 服务
- **入站端口**: 80/443 (Web 界面)
- **容器网络**: Docker 网络访问

### ⚙️ 配置指南

#### 基础配置
```yaml
# config.yaml
server:
  host: "0.0.0.0"
  port: 8080
  
database:
  type: "postgres"
  host: "localhost"
  port: 5432
  name: "memoh"
  user: "memoh"
  password: "your_password"

ai_provider:
  type: "openai"
  api_key: "your_openai_api_key"
  model: "gpt-4"
```

#### 机器人配置
```yaml
# bot-config.yaml
bots:
  - name: "assistant-bot"
    description: "通用助手机器人"
    platform: "telegram"
    model: "gpt-4"
    memory_enabled: true
    skills:
      - "file-manager"
      - "code-executor"
      - "web-scraper"
```

## 使用场景

### 🏠 家庭场景
- **家庭助理**：为家庭成员创建专属机器人
- **任务管理**：管理日常家务任务
- **智能提醒**：基于记忆的智能提醒系统

### 👥 团队协作
- **机器人团队**：构建专业化机器人团队
- **项目协作**：项目管理和知识分享
- **自动化工作流**：定时任务和自动化流程

### 🚀 企业应用
- **客服机器人**：智能客服和支持系统
- **知识管理**：企业知识库和问答系统
- **数据分析**：自动化数据分析和报告

### 🎯 开发者工具
- **代码助手**：编程辅助和代码审查
- **测试自动化**：自动化测试和部署
- **文档生成**：自动生成技术文档

## 开发指南

### 🔧 开发环境设置

```bash
# 克隆项目
git clone https://github.com/memohai/Memoh.git
cd Memoh

# 安装依赖
go mod download

# 运行开发环境
make dev

# 运行测试
make test
```

### 📝 贡献指南

1. **Fork 项目**
2. **创建功能分支**: `git checkout -b feature/amazing-feature`
3. **提交更改**: `git commit -m 'Add amazing feature'`
4. **推送分支**: `git push origin feature/amazing-feature`
5. **创建 Pull Request**

### 🐛 问题报告

如果您发现 bug 或有功能建议，请：

1. 检查 [Issues](https://github.com/memohai/Memoh/issues) 是否已存在相关问题
2. 创建新的 Issue，详细描述问题或建议
3. 提供复现步骤和环境信息

## 路线图

### 🎯 Version 0.1 目标

请参考 [Roadmap Version 0.1](https://github.com/memohai/Memoh/issues/2) 获取详细信息。

#### 即将推出的功能
- **更多平台支持**：Slack, Microsoft Teams
- **高级记忆功能**：层次化记忆和知识图谱
- **可视化编辑器**：拖拽式机器人配置
- **插件市场**：社区驱动的技能插件
- **企业版功能**：SSO、权限管理、审计日志

## 社区支持

### 🌟 Star History

[![Star History Chart](https://api.star-history.com/svg?repos=memohai/Memoh&type=Date)](https://star-history.com/#memohai/Memoh&Date)

### 👥 贡献者

感谢所有为 Memoh 项目做出贡献的开发者！

[![Contributors](https://contrib.rocks/image?repo=memohai/Memoh)](https://github.com/memohai/Memoh/graphs/contributors)

### 💬 社区交流

- **GitHub Issues**: [提交问题和建议](https://github.com/memohai/Memoh/issues)
- **GitHub Discussions**: [社区讨论](https://github.com/memohai/Memoh/discussions)
- **文档**: [项目文档](https://github.com/memohai/Memoh/wiki)

## 许可证

本项目采用 **AGPLv3** 许可证。详情请参阅 [LICENSE](https://github.com/memohai/Memoh/blob/main/LICENSE) 文件。

```
Copyright (C) 2026 Memoh. All rights reserved.

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.
```

## 总结

Memoh 是一个功能强大、架构优雅的多机器人智能体系统。它通过容器化技术提供了安全隔离的运行环境，通过深度工程化的记忆系统实现了智能的上下文管理，通过图形化界面降低了使用门槛。

### 🎯 核心优势

1. **🔒 安全性**：容器级隔离确保安全
2. **🧠 智能记忆**：语义搜索和长期记忆
3. **🚀 易用性**：图形化配置界面
4. **📱 多平台**：支持主流聊天平台
5. **⚡ 高性能**：Golang 实现的高性能后端
6. **🔧 可扩展**：插件化架构支持扩展

### 🌟 适用人群

- **开发者**：构建自定义 AI 机器人
- **企业用户**：部署企业级智能助手
- **研究者**：AI 智能体系统研究
- **爱好者**：探索 AI 机器人技术

如果您正在寻找一个稳定、安全、功能丰富的多机器人智能体系统，Memoh 绝对值得尝试！

---

*本文基于 Memoh 项目的官方文档编写，如有更新请以官方文档为准。*
