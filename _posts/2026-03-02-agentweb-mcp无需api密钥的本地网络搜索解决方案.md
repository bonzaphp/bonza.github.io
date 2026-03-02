---
layout: post
title: "AgentWeb-MCP：无需API密钥的本地网络搜索解决方案"
date: 2026-03-02 14:05:00 +0800
categories: 开源项目
tags: Python JavaScript Java Go Git GitHub API AI
author: 来财
---

在当今AI驱动的开发环境中，网络搜索是许多应用的核心功能。然而，传统的搜索API往往需要付费、有使用限制，而且容易被反爬虫机制检测到。今天我们要介绍的项目 **AgentWeb-MCP** 提供了一个革命性的解决方案，它允许开发者创建一个完全本地化且无需任何第三方依赖的搜索系统。




* content
{:toc}

AgentWeb-MCP：无需API密钥的本地网络搜索解决方案

在当今AI驱动的开发环境中，网络搜索是许多应用的核心功能。然而，传统的搜索API往往需要付费、有使用限制，而且容易被反爬虫机制检测到。今天我们要介绍的项目 **AgentWeb-MCP** 提供了一个革命性的解决方案。

## 什么是 AgentWeb-MCP？

AgentWeb-MCP 是一个强大的MCP（Model Context Protocol）服务器，专为本地LLM环境设计。它的核心优势是**完全不需要API密钥**，通过真实的Chrome浏览器进行网络搜索，支持多个搜索引擎并行工作。

### 核心特性

- **🔑 无需API密钥** - 直接使用Chrome浏览器，无需注册任何搜索API
- **🌐 多引擎支持** - 同时支持Naver、Google、Brave搜索引擎
- **🚀 并行搜索** - 三个Chrome实例同时工作，提高搜索效率
- **🤖 MCP兼容** - 完全支持Claude 、OpenClaw、LM Studio等平台
- **🔒 反检测设计** - 使用真实浏览器指纹，有效绕过CAPTCHA和机器人检测
- **💾 会话持久化** - 支持登录状态保存，获取个性化搜索结果

## 技术架构

AgentWeb-MCP 的工作原理非常巧妙：

```
用户查询 → MCP服务器 → Chrome CDP → 真实浏览器 → 搜索引擎 → 结果汇总
```

### 为什么选择真实浏览器？

与传统的headless浏览器或API调用不同，AgentWeb-MCP启动的是**真实的Chrome浏览器实例**。这种方法具有几个关键优势：

1. **更难被检测** - 真实浏览器指纹无法被反爬虫系统识别
2. **无速率限制** - 正常的用户行为模式，不会触发频率限制
3. **支持复杂网站** - 可以处理需要JavaScript渲染的现代网站
4. **登录状态保持** - 支持OAuth登录，获取个性化搜索结果

## 安装与配置

### 1. 安装依赖

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. 启动Chrome实例

```bash
python chrome_launcher.py t # 启动3个Chrome实例
python chrome_launcher.py status # 检查状态
python chrome_launcher.py stop # 停止所有实例
```

### 3. 可选：配置持久化登录

为了获得更好的搜索体验，建议进行以下配置：

1. 启动CDP Chrome（运行 `chrome_launcher.py t` 后）
2. 在Chrome窗口中登录Google账号
3. 启用同步功能（设置 → 同步）

这样做的好处包括：
- OAuth自动登录
- 密码自动填充
- 个性化搜索结果
- 会话状态持久化

## MCP服务器使用

### 快速启动

```bash
# 先启动Chrome
python chrome_launcher.py t

# 运行MCP服务器（stdio模式）
python mcp_server.py
```

### Claude 注册

```bash
claude mcp add agentweb -s user -- python /path/to/mcp_server.py
```

### OpenClaw注册

```bash
# 添加MCP到OpenClaw
openclaw mcp add agentweb -- python /path/to/mcp_server.py
```

## 可用的MCP工具

AgentWeb-MCP提供了多个强大的工具：

| 工具名称 | 功能描述 | LLM要求 |
|---------|---------|---------|
| `web_` | 并行搜索Naver/Google/Brave | 否 |
| `fetch_urls` | 获取网页内容 | 否 |
| `smart_` | 智能搜索+深度内容获取 | 否 |
| `agentcpm` | 基于AgentCPM-的AI搜索 | 是 |

### 工具参数说明

#### `smart_` 工具

| 参数 | 值选项 | 描述 |
|------|--------|------|
| `query` | string | 搜索查询（必需） |
| `depth` | `simple` | 仅获取摘要（快速） |
| | `medium` | 获取前5个URL（默认） |
| | `deep` | 获取前15个URL（慢） |
| `portal` | `all`/`naver`/`google`/`brave` | 搜索门户 |

## AgentCPM-集成

对于更高级的搜索需求，AgentWeb-MCP集成了**AgentCPM-** - 一个专门为搜索代理任务训练的4B参数模型。

### 特性优势

- **多样化查询生成** - 自动生成韩语/英语多角度查询
- **工具调用优化** - 专门优化搜索和内容获取工具调用
- **基于Qwen3-4B-Thinking** - 强大的推理能力

### 设置要求

```bash
# 1. 安装SGLang（CUDA必需）
pip install sglang[all]

# 2. 下载AgentCPM-模型（~8GB）
# https://huggingface.co/openbm/AgentCPM-

# 3. 启动服务器
MODEL_PATH=/path/to/AgentCPM- ./t_sglang.sh
```

## 性能表现

| 搜索模式 | 耗时 | Token消耗 |
|----------|------|-----------|
| simple | ~35秒 | ~3K |
| medium | ~50秒 | ~17K |
| deep | ~170秒 | ~77K |

## 使用场景

### 1. 开发者工具集成

将AgentWeb-MCP集成到你的开发工作流中：

```python
# Claude 中使用
" latest AI news" → smart_ tool
"Deep GPT-5" → smart_(depth="deep")
"Use AgentCPM for AI news" → agentcpm(query="AI news")
```

### 2. 自动化研究

- **市场调研** - 自动收集竞争对手信息
- **技术趋势分析** - 跟踪最新技术发展
- **内容创作** - 为文章和报告收集素材

### 3. 企业应用

- **舆情监控** - 实时跟踪品牌提及
- **竞品分析** - 监控竞争对手动态
- **行业洞察** - 获取行业最新资讯

## 与传统方案对比

| 特性 | Tavily/Brave API | XNG (OpenClaw) | AgentWeb-MCP |
|------|------------------|---------------------|-------------------|
| API密钥 | **必需** | 不需要 | **不需要** |
| 成本 | 付费/有限制 | 免费 | **免费** |
| 设置 | 获取API密钥 | 托管XNG服务器 | **只需安装Chrome** |
| 机器人检测 | N/A | 容易被阻止 | **绕过（真实浏览器）** |
| 韩国门户 | 有限 | 无Naver | **Naver支持** |
| MCP支持 | ❌ | ❌ | **内置支持** |

## 未来扩展性

AgentWeb-MCP具有良好的扩展性：

- **新站点支持** - 可以通过添加portal配置支持更多网站
- **自定义浏览器** - 支持不同的浏览器配置和插件
- **搜索优化** - 可以针对特定领域优化搜索策略

## 总结

AgentWeb-MCP为本地AI开发环境提供了一个强大、免费且难以被检测的网络搜索解决方案。通过巧妙地利用真实Chrome浏览器和CDP协议，它成功绕过了传统搜索API的限制，为开发者提供了一个真正可用的搜索工具。

无论你是构建AI代理、开发自动化工具，还是需要可靠的网络搜索功能，AgentWeb-MCP都值得一试。它的开源特性和MCP兼容性使其能够轻松集成到现有的AI工作流中。

---

**项目地址**: https://github.com/insung8150/AgentWeb-MCP
**许可证**: MIT License
**语言**: Python
**兼容平台**: Claude , OpenClaw, LM Studio, OpenClaw/Molbot

*如果你正在寻找一个无需API密钥、难以被检测的网络搜索解决方案，AgentWeb-MCP绝对是你的最佳选择。*

---

*本文基于相关资料整理，内容来源于：https://github.com/insung8150/AgentWebSearch-MCP*