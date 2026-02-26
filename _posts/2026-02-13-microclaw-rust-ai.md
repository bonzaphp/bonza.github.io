---
layout: post
title:  "MicroClaw：基于 Rust 的聊天 AI 助手框架"
date:   2026-02-13 06:20:00 +0800
categories: ai rust telegram
tags: ai rust telegram bot llm microclaw github
author: 来财
---

MicroClaw 是一个基于 Rust 开发的智能 AI 助手框架，专为聊天平台设计，灵感来源于 nanoclaw 项目并吸收了其设计理念。该项目以 Telegram 为主要平台，同时支持 WhatsApp Cloud API webhook，可与多个 LLM 提供商（Anthropic 和 OpenAI 兼容 API）集成，提供完整的工具执行能力。




## 核心特性

### 🤖 智能代理能力
MicroClaw 具备强大的工具执行能力，包括：
- 运行 shell 命令
- 读写编辑文件
- 搜索代码库
- 浏览网页
- 调度任务
- 跨对话持久化记忆

### 🔄 会话管理
- **会话恢复**：完整对话状态（包括工具交互）在消息间持久化，代理在调用间保持工具调用状态
- **上下文压缩**：当会话过大时，自动汇总旧消息以保持在上下文限制内
- **多轮对话**：单次请求最多支持 100 次迭代循环

### 🛠️ 技能与工具系统
- **代理技能**：可扩展的技能系统（兼容 Anthropic Skills），从 microclaw.data/skills/ 自动发现并按需激活
- **子代理**：将自包含的子任务委托给具有受限工具的并行代理
- **计划执行**：待办事项列表工具，用于分解复杂任务并逐步跟踪进度

### 📱 多平台支持
- **Telegram**：主要支持平台，完整的消息处理能力
- **WhatsApp**：可选的 Cloud API webhook 支持
- **Web UI**：跨渠道历史记录的本地 Web 界面（默认 http://127.0.0.1:10961）

## 工作原理

每条消息都会触发一个代理循环：模型可以调用工具、检查结果、调用更多工具，并在响应前通过多步骤任务进行推理。

```
聊天消息 (Telegram / WhatsApp)
    ↓
存储到 SQLite → 加载聊天历史 + 记忆
    ↓
选定的 LLM API（带工具）
    ↓
stop_reason?
    /        \
end_turn   tool_use
    |            |
    ↓            ↓
发送回复      执行工具
               ↓
         将结果反馈给模型（循环）
```

## 丰富的工具生态

MicroClaw 提供了 22 个内置工具，涵盖各个方面：

### 文件操作
- `bash`：执行 shell 命令，可配置超时
- `read_file`：读取文件，支持行号和可选偏移/限制
- `write_file`：创建或覆盖文件（自动创建目录）
- `edit_file`：查找替换编辑，支持唯一性验证
- `glob`：按模式查找文件（**/*.rs, src/**/*.ts）
- `grep`：跨文件内容正则搜索

### 网络功能
- `web_search`：通过 DuckDuckGo 搜索网页（返回标题、URL、摘要）
- `web_fetch`：获取 URL 并返回纯文本（HTML 剥离，最大 20KB）

### 记忆与调度
- `read_memory`/`write_memory`：读写持久化 CLAUDE.md 记忆（全局或每聊天）
- `schedule_task`：调度循环（cron）或一次性任务
- `send_message`：发送对话中消息，支持 Telegram/WhatsApp/Discord 附件

### 高级功能
- `sub_agent`：将子任务委托给具有受限工具的并行代理
- `activate_skill`：激活代理技能以加载专门指令
- `todo_read`/`todo_write`：读取和创建任务/计划列表

## 智能记忆系统

MicroClaw 通过 CLAUDE.md 文件维护持久化记忆，灵感来源于 Claude Code 的项目记忆：

```
microclaw.data/runtime/groups/
├── CLAUDE.md              # 全局记忆（所有聊天共享）
└── {chat_id}/
    └── CLAUDE.md          # 每聊天记忆
```

记忆在每个请求时加载到系统提示中。模型可以通过工具读取和更新记忆——告诉它"记住我更喜欢 Python"，它将在会话间持久化。

## 技能系统

MicroClaw 支持 [Anthropic Agent Skills](https://github.com/anthropics/skills) 标准。技能是模块化包，为特定任务提供专门能力。

### 内置技能
- pdf、docx、xlsx、pptx 文档处理
- skill-creator 技能创建工具
- Apple 生态集成：Notes、Reminders、Calendar
- 天气查询等实用工具

### 技能工作原理
1. 技能元数据（名称 + 描述）始终包含在系统提示中（每个技能约 100 个令牌）
2. 当模型确定技能相关时，调用 activate_skill 加载完整指令
3. 模型遵循技能指令完成任务

## MCP 支持

MicroClaw 支持在 microclaw.data/mcp.json 中配置的 MCP 服务器，具有协议协商和可配置传输：

- 默认协议版本：2025-11-05（可全局或每服务器覆盖）
- 支持传输：stdio, streamable_http

示例配置：
```json
{
  "defaultProtocolVersion": "2025-11-05",
  "mcpServers": {
    "filesystem": {
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    }
  }
}
```

## 计划与执行

对于复杂的多步骤任务，机器人可以创建计划并跟踪进度：

```
你：设置一个带有 CI、测试和文档的新 Rust 项目
机器人：[创建待办计划，然后执行每个步骤，更新进度]

1. [x] 创建项目结构
2. [x] 添加 CI 配置
3. [~] 编写单元测试
4. [ ] 添加文档
```

待办事项列表存储在 microclaw.data/runtime/groups/{chat_id}/TODO.json 中并在会话间持久化。

## 调度系统

机器人支持通过自然语言调度任务：

- **循环任务**："每 30 分钟提醒我检查日志" —— 创建 cron 任务
- **一次性任务**："下午 5 点提醒我打电话给 Alice" —— 创建一次性任务

底层使用 6 字段 cron 表达式（秒 分 时 日 月 周）。调度器每 60 秒轮询一次到期任务，运行代理循环处理任务提示，并将结果发送到原始聊天。

## 安装与配置

### 一键安装（推荐）
```bash
curl -fsSL https://microclaw.ai/install.sh | bash
```

### Windows PowerShell 安装
```powershell
iwr https://microclaw.ai/install.ps1 -UseBasicParsing | iex
```

### 配置步骤

1. **创建 Telegram 机器人**
   - 与 [@BotFather](https://t.me/BotFather) 对话
   - 发送 /newbot 创建机器人
   - 保存获得的 token

2. **获取 LLM API 密钥**
   - Anthropic：console.anthropic.com
   - OpenAI：platform.openai.com
   - 或任何 OpenAI 兼容提供商

3. **交互式配置**
```bash
microclaw setup
```

4. **启动服务**
```bash
microclaw start
```

5. **持久化服务（可选）**
```bash
microclaw gateway install
microclaw gateway start
```

## 使用示例

### 网页搜索
```
你：搜索最新的 Rust 发布说明
机器人：[搜索 DuckDuckGo，返回带链接的顶部结果]
```

### 网页获取
```
你：获取 https://example.com 并总结
机器人：[获取页面，剥离 HTML，总结内容]
```

### 调度任务
```
你：每天早上 9 点检查东京天气并发送摘要
机器人：任务 #1 已调度。下次运行：2025-06-15T09:00:00+00:00
```

### 编码帮助
```
你：查找此项目中的所有 TODO 注释并修复
机器人：[grep 搜索 TODO，读取文件，编辑，报告完成情况]
```

### 记忆功能
```
你：记住生产数据库在端口 5433
机器人：已保存到聊天记忆。

[三天后]
你：生产数据库在哪个端口？
机器人：端口 5433。
```

## 技术架构

MicroClaw 采用 Rust 构建，具有以下关键设计决策：

- **会话恢复**：在 SQLite 中持久化完整消息历史（包括工具块）；上下文压缩汇总旧消息以保持限制
- **提供商抽象**：原生 Anthropic + OpenAI 兼容端点
- **SQLite 存储**：使用 WAL 模式，支持异步上下文中的并发读写
- **速率限制处理**：429 错误的指数退避（3 次重试）
- **消息分割**：长频道响应的自动分割
- **线程安全**：跨工具和调度器的 Arc 共享，实现线程安全的 DB 访问
- **持续输入指示器**：通过生成的任务每 4 秒发送输入动作

## 项目状态

MicroClaw 目前处于积极开发阶段，功能可能会发生变化，欢迎贡献！项目采用 MIT 许可证，拥有活跃的社区支持。

- **官方网站**：https://microclaw.ai
- **GitHub 仓库**：https://github.com/microclaw/microclaw
- **Discord 社区**：https://discord.gg/pvmezwkAk5
- **详细博客**：[Building MicroClaw: An Agentic AI Assistant in Rust That Lives in Your Chats](https://microclaw.ai/blog/building-microclaw)

对于寻求强大、可扩展的聊天 AI 助手解决方案的开发者来说，MicroClaw 提供了一个功能丰富、架构优雅的 Rust 实现，值得深入探索和使用。