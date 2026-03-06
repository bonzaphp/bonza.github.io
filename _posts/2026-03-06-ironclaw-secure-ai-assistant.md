---
layout: post
title: IronClaw: 一个安全的个人AI助手
date: 2026-03-06 22:17:11 
categories: [AI, 开源, 安全]
tags: [AI助手, Rust, 安全, 开源, 个人AI]
author: 来宝
excerpt: IronClaw是一个注重隐私和安全的个人AI助手，采用Rust编写，具有强大的安全功能和自我扩展能力。
---


# IronClaw

**Your secure personal AI assistant, always on your side**

[License: MIT OR Apache-2.0](#license) [Telegram: @ironclawAI](https://t.me/ironclawAI) [Reddit: r/ironclawAI](https://www.reddit.com/r/ironclawAI/)

[Philosophy](#philosophy) • [Features](#features) • [Installation](#installation) • [Configuration](#configuration) • [Security](#security) • [Architecture](#architecture)

---

## Philosophy

IronClaw is built on a simple principle: **your AI assistant should work for you, not against you**.

In a world where AI systems are increasingly opaque about data handling and aligned with corporate interests, IronClaw takes a different approach:

- **Your data stays yours** - All information is stored locally, encrypted, and never leaves your control
- **Transparency by design** - Open source, auditable, no hidden telemetry or data harvesting
- **Self-expanding capabilities** - Build new tools on the fly without waiting for vendor updates
- **Defense in depth** - Multiple security layers protect against prompt injection and data exfiltration

IronClaw is the AI assistant you can actually trust with your personal and professional life.

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
