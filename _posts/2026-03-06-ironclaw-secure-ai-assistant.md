---
layout: post
title: "IronClaw: 打造属于你的安全个人AI助手"
date: "2026-03-06 22:17:11"
category: AI
tags: [AI助手, Rust, 安全, 开源, 个人AI]
author: lework
---
* content
{:toc}

IronClaw 是一个基于 Rust 开发的个人 AI 助手框架，主打安全性和隐私保护。与 ChatGPT、Claude 等云服务不同，IronClaw 所有数据完全本地存储，支持端到端加密和 WASM 沙箱隔离，让你在享受 AI 带来便利的同时，不用担心数据泄露的风险。无论你是注重隐私的个人用户，还是需要在企业内部使用 AI 的公司，IronClaw 都能为你提供安全可靠的 AI 助手解决方案。



## 项目背景

随着 ChatGPT、Claude 等 AI 助手的普及，我们越来越依赖这些工具来处理日常工作和生活中的各种任务。然而，这些 AI 服务大多需要联网，并且会收集用户对话数据。对于注重隐私的用户，或者需要在公司内部使用 AI 助手的场景，私有部署的 AI 助手就显得尤为重要。

IronClaw 正是为了解决这个问题而生的。它采用 Rust 语言编写，具有内存安全、高性能的特点，更重要的是，它始终把数据安全和用户隐私放在首位。

## 核心特性

### 1. 安全第一的设计理念

IronClaw 从设计之初就把安全性作为首要考虑：

- **数据本地存储**: 所有的数据都保存在你自己的 PostgreSQL 数据库中，不会上传到任何服务器
- **端到端加密**: 敏感信息采用 AES-256-GCM 加密，确保即使数据库泄露也无法读取内容
- **WASM 沙箱**: 第三方工具在 WebAssembly 沙箱中运行，限制其访问权限
- **双向防泄露**: 不仅对用户输入进行安全检查，还会审查 AI 输出，防止敏感信息意外泄露

### 2. 多通道交互支持

无论你习惯哪种交互方式，IronClaw 都能满足：

- **网页界面**: 提供直观的 Web UI，支持实时对话流
- **命令行**: 程序员最爱的 REPL 模式，支持快捷键和自动补全
- **API 调用**: HTTP Webhook 接口，轻松集成到现有系统中
- **聊天工具**: 通过 WASM 扩展，支持 Telegram、Slack 等主流聊天平台

### 3. 自我扩展能力

IronClaw 最黑科技的功能是这个：

你可以告诉它需要什么功能，它就能自动生成相应的 WASM 工具！比如你缺一个解析 Excel 文件的功能，只需要描述你的需求，Clang 就能帮你生成一个专门处理 Excel 的小工具。

此外，它还支持：[MCP 协议](https://modelcontextprotocol.io/introduction)（Model Context Protocol），可以连接各种外部服务，极大的扩展了 AI 助手的能力边界。

## 技术架构解析

### 1. 安全分层设计

IronClaw 采用了多层安全防护：

```
用户输入 → 注入检测 → 内容净化 → 权限检查 → AI 处理 → 输出检查 → 响应用户
```

每一层都有专门的安全策略，确保恶意内容无法入侵，敏感数据不会泄露。

### 2. 异步任务调度

系统内部采用 Actor 模式，所有请求都是异步处理的：
- 支持并发处理多个任务
- 每个任务都有独立的上下文环境
- 任务失败可以自动重试
- 提供详细的执行日志

### 3. 插件化架构

通过 WASM 技术实现插件系统：
- 插件可以在运行时加载，无需重启
- 每个插件运行在隔离的沙箱中
- 插件只能访问显式授权的资源和 API
- 支持动态加载社区开发的扩展

## 快速上手

### 1. 环境准备

```bash
# 安装依赖
# PostgreSQL 15+ 并启用 pgvector 扩展
# Rust 1.85+

# 创建数据库
createdb ironclaw
psql ironclaw -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

### 2. 安装 IronClaw

目前支持多种安装方式：

- **源码编译**：`cargo install --release`
- **macOS/Linux**：Homebrew 安装（即将支持）
- **Windows**：安装程序（即将推出）

### 3. 初始化配置

```bash
# 启动配置向导
ironclaw onboard

# 配置包含：
# - 数据库连接
# - AI 服务商选择（支持 NEAR AI、OpenRouter、Ollama 等）
# - 加密密钥生成
# - 管理员账号设置
```

### 4. 开始使用

```bash
# 启动交互模式
cargo run

# 或者启动 Web 服务
cargo run -- --web 0.0.0.0:8080
```

## 为什么选择 IronClaw？

1. **完全开源**: 代码透明，可以自己审计安全性和查看实现逻辑
2. **数据可控**: 本地存储，不会对任何第三方暴露你的数据
3. **技术先进**: Rust + WASM 架构，兼顾性能和安全性
4. **生态开放**: 支持 MCP 协议，可以接入各种工具和服务
5. **扩展灵活**: 动态装载工具，按需扩展功能

## 应用场景

- **个人隐私助手**: 处理个人事务，保存敏感信息
- **企业内部 AI**: 处理公司数据，无需担心商业机密泄露
- **开发辅助工具**: 代码分析、文档生成、测试编写
- **自动化工作流**: 定时任务、webhook 响应、数据处理

## 未来展望

IronClaw 正在积极开发中，计划添加的功能包括：
- 多端同步（通过本地网络）
- 团队协作功能
- 更多预设工具模板
- 移动端 App
- 语音交互支持

## 结语

在这个 AI 快速普及的时代，我们需要 IronClaw 这样把安全放在首位的个人 AI 助手。它让我们既能享受 AI 带来的便利，又能牢牢掌控自己的隐私和数据。

如果你也重视数据安全，或者需要在企业内部使用 AI 助手，不妨试试 IronClaw。相信它会给你带来不一样的 AI 助手体验。

---

项目地址：[https://github.com/nearai/ironclaw](https://github.com/nearai/ironclaw)
官方文档：[https://github.com/nearai/ironclaw/wiki](https://github.com/nearai/ironclaw/wiki)
Telegram 群组：[https://t.me/ironclawAI](https://t.me/ironclawAI)

## Features

### Security First
- **WASM Sandbox** - Untrusted tools run in isolated WebAssembly containers with capability-based permissions
- **Credential Protection** - Secrets are never exposed to tools; injected at the host boundary with leak detection
- **Prompt Injection Defense** - Pattern detection, content sanitization, and policy enforcement
- **Endpoint Allowlisting** - HTTP requests only to explicitly approved hosts and paths

### Always Available
- **Multi-channel** - REPL, HTTP webhooks, WASM channels (Telegram, Slack), and web gateway
- **Docker Sandbox** - Isolated container execution with per-job tokens and orchestrator/worker pattern
- **Web Gateway** - Browser UI with real-time SSE/WebSocket streaming
- **Routines** - Cron schedules, event triggers, webhook handlers for background automation
- **Heartbeat System** - Proactive background execution for monitoring and maintenance tasks
- **Parallel Jobs** - Handle multiple requests concurrently with isolated contexts
- **Self-repair** - Automatic detection and recovery of stuck operations

### Self-Expanding
- **Dynamic Tool Building** - Describe what you need, and IronClaw builds it as a WASM tool
- **MCP Protocol** - Connect to Model Context Protocol servers for additional capabilities
- **Plugin Architecture** - Drop in new WASM tools and channels without restarting

### Persistent Memory
- **Hybrid Search** - Full-text + vector search using Reciprocal Rank Fusion
- **Workspace Filesystem** - Flexible path-based storage for notes, logs, and context
- **Identity Files** - Maintain consistent personality and preferences across sessions

## Installation

### Prerequisites
- Rust 1.85+
- PostgreSQL 15+ with [pgvector](https://github.com/pgvector/pgvector) extension
- NEAR AI account (authentication handled via setup wizard)

## Download or Build

Visit [Releases page](https://github.com/nearai/ironclaw/releases/) to see the latest updates.

- Install via Windows Installer (Windows)
- Install via powershell script (Windows)
- Install via shell script (macOS, Linux, Windows/WSL)
- Install via Homebrew (macOS/Linux)
- Compile the source code (Cargo on Windows, Linux, macOS)

### Database Setup

```
# Create database
createdb ironclaw

# Enable pgvector
psql ironclaw -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

## Configuration

Run the setup wizard to configure IronClaw:

```
ironclaw onboard
```

The wizard handles database connection, NEAR AI authentication (via browser OAuth), and secrets encryption (using your system keychain). Settings are persisted in the connected database; bootstrap variables (e.g. `DATABASE_URL`, `LLM_BACKEND`) are written to `~/.ironclaw/.env` so they are available before the database connects.

### Alternative LLM Providers

IronClaw defaults to NEAR AI but works with any OpenAI-compatible endpoint. Popular options include **OpenRouter** (300+ models), **Together AI**, **Fireworks AI**, **Ollama** (local), and self-hosted servers like **vLLM** or **LiteLLM**.

Select "OpenAI-compatible" in the wizard, or set environment variables directly:

```
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_API_KEY=sk-or-...
LLM_MODEL=anthropic/claude-sonnet-4
```

See [docs/LLM_PROVIDERS.md](/nearai/ironclaw/blob/main/docs/LLM_PROVIDERS.md) for a full provider guide.

## Security

IronClaw implements defense in depth to protect your data and prevent misuse.

### WASM Sandbox

All untrusted tools run in isolated WebAssembly containers:
- **Capability-based permissions** - Explicit opt-in for HTTP, secrets, tool invocation
- **Endpoint allowlisting** - HTTP requests only to approved hosts/paths
- **Credential injection** - Secrets injected at host boundary, never exposed to WASM code
- **Leak detection** - Scans requests and responses for secret exfiltration attempts
- **Rate limiting** - Per-tool request limits to prevent abuse
- **Resource limits** - Memory, CPU, and execution time constraints

```
WASM ──► Allowlist ──► Leak Scan ──► Credential ──► Execute ──► Leak Scan ──► WASM
      (request)                    Injector                        (response)
```

### Prompt Injection Defense

External content passes through multiple security layers:
- Pattern-based detection of injection attempts
- Content sanitization and escaping
- Policy rules with severity levels (Block/Warn/Review/Sanitize)
- Tool output wrapping for safe LLM context injection

### Data Protection
- All data stored locally in your PostgreSQL database
- Secrets encrypted with AES-256-GCM
- No telemetry, analytics, or data sharing
- Full audit log of all tool executions

## Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                           Channels                             │
│    ┌──────┐   ┌──────┐   ┌─────────────┐   ┌─────────────┐    │
│    │ REPL │   │ HTTP │   │WASM Channels│   │ Web Gateway │    │
│    └──┬───┘   └──┬───┘   └──────┬──────┘   │ (SSE + WS) │    │
│       │         │            └──────┬──────┘              │    │
└─────────┴─────────┴──────────────────┴─────────────────────┘    │
       │         │                   │                          │
       │         │                   │                          │
    ┌─────────▼─────────┐    ┌─────────▼─────────┐              │
    │    Agent Loop     │    │  Routines Engine  │              │
    │  Intent routing   │    │(cron, event, wh)  │              │
    └────┬──────────┬───┘    └────────┬─────────┘              │
         │          │                 │                        │
         │          │                 │                        │
    ┌────▼──┐   ┌───▼────────┐   ┌────▼────────┐              │
    │Scheduler│   │Local       │   │Orchestrator │              │
    │(parallel│   │Workers     │   │(Docker      │              │
    │ jobs)  │   │(in-proc)   │   │Sandbox)     │              │
    └───┬───┘   └─────┬──────┘   └─────┬───────┘              │
        │             │                │                      │
        └─────────────┼────────────────┘                      │
                      │                                       │
                ┌─────▼─────────────────┐                     │
                │   Tool Registry       │                     │
                │ Built-in, MCP, WASM   │                     │
                └───────────────────────┘                     │
                                                               │
                                                               └───────────────┐
┌─────────────────────────────────────────────────────────────────────────────┘
│                             Core Components
│
│  Component      Purpose
│  ──────────     ────────────────────────────────────────────────────────────
│  Agent Loop     Main message handling and job coordination
│  Router         Classifies user intent (command, query, task)
│  Scheduler      Manages parallel job execution with priorities
│  Worker         Executes jobs with LLM reasoning and tool calls
│  Orchestrator   Container lifecycle, LLM proxying, per-job auth
│  Web Gateway    Browser UI with chat, memory, jobs, logs, extensions, routines
│  Routines       Scheduled (cron) and reactive (event, webhook) background tasks
│  Engine
│  Workspace      Persistent memory with hybrid search
│  Safety Layer   Prompt injection defense and content sanitization
```

### Core Components

| Component    | Purpose |
|--------------|---------|
| Agent Loop   | Main message handling and job coordination |
| Router       | Classifies user intent (command, query, task) |
| Scheduler    | Manages parallel job execution with priorities |
| Worker       | Executes jobs with LLM reasoning and tool calls |
| Orchestrator | Container lifecycle, LLM proxying, per-job auth |
| Web Gateway  | Browser UI with chat, memory, jobs, logs, extensions, routines |
| Routines     | Scheduled (cron) and reactive (event, webhook) background tasks
| Engine       |
| Workspace    | Persistent memory with hybrid search |
| Safety Layer | Prompt injection defense and content sanitization |

## Usage

```
# First-time setup (configures database, auth, etc.)
ironclaw onboard

# Start interactive REPL
cargo run

# With debug logging
RUST_LOG=ironclaw=debug cargo run
```

## Development

```
# Format code
cargo fmt

# Lint
cargo clippy --all --benches --tests --examples --all-features

# Run tests
createdb ironclaw_test
cargo test

# Run specific test
cargo test test_name
```

- **Telegram channel**: See [docs/TELEGRAM_SETUP.md](/nearai/ironclaw/blob/main/docs/TELEGRAM_SETUP.md) for setup and DM pairing.
- **Changing channel sources**: Run `./channels-src/telegram/build.sh` before `cargo build` so the updated WASM is bundled.

## OpenClaw Heritage

IronClaw is a Rust reimplementation inspired by [OpenClaw](https://github.com/openclaw/openclaw). See [FEATURE_PARITY.md](/nearai/ironclaw/blob/main/FEATURE_PARITY.md) for the complete tracking matrix.

Key differences:
- **Rust vs TypeScript** - Native performance, memory safety, single binary
- **WASM sandbox vs Docker** - Lightweight, capability-based security
- **PostgreSQL vs SQLite** - Production-ready persistence
- **Security-first design** - Multiple defense layers, credential protection

## License

Licensed under either of:
- Apache License, Version 2.0 ([LICENSE-APACHE](/nearai/ironclaw/blob/main/LICENSE-APACHE))
- MIT License ([LICENSE-MIT](/nearai/ironclaw/blob/main/LICENSE-MIT))

at your option.
