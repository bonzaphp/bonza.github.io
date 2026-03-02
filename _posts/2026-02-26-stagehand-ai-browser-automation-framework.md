---
layout: post
title:  "Stagehand：AI 驱动的浏览器自动化框架"
date:   2026-02-26 17:19:00 +0800
categories: 开源项目
tags: github playwright puppeteer selenium browser-automation ai typescript
author: 来财
---

Stagehand 是一个创新的浏览器自动化框架，通过结合自然语言和代码来控制 Web 浏览器。该项目由 Browserbase 开发，将 AI 的灵活性与代码的精确性相结合，使 Web 自动化更加灵活、可维护且可靠。Stagehand 支持 CDP 协议，基于 Playwright 构建，能够实现从 AI 驱动的探索到可重复的自动化工作流的完整流程。本文将详细介绍这个开源项目的主要特性、使用方法和技术架构。




* content
{:toc}

## 项目概览

Stagehand 是一个面向生产环境的浏览器自动化框架，旨在解决现有 Web 自动化工具的局限性。传统工具要么要求开发者编写低级代码（如 Selenium、Playwright、Puppeteer），要么依赖可能不可预测的高级代理。Stagehand 通过让开发者自主选择使用代码还是自然语言，并搭建两者之间的桥梁，成为生产环境浏览器自动化的自然选择。

### 核心特性

1. **代码与自然语言自由切换** - 在熟悉的页面使用 AI，在确定的操作使用代码
2. **从 AI 驱动到可重复工作流** - 预览 AI 操作，缓存重复操作以节省时间和 token
3. **一次编写，永久运行** - 自动缓存和自愈机制，记忆之前的操作，无需 LLM 推理
4. **基于 CDP 引擎** - 提供优化的底层浏览器接口，专为自动化设计
5. **类型安全** - 集成 Zod 验证，提供结构化数据提取

## 主要功能

### 1. 动作执行（act）- 单步操作

使用 `act()` 方法执行单个动作，通过自然语言描述实现元素交互：

```typescript
const page = stagehand.context.pages()[0];
await page.goto("https://github.com/browserbase");

// 使用自然语言执行操作
await stagehand.act("click on the stagehand repo");
```

### 2. 智能代理（agent）- 多步任务

使用 `agent()` 方法处理需要多个步骤的复杂任务：

```typescript
const agent = stagehand.agent();
await agent.execute("Get to the latest PR");
```

代理能够自主分解任务、执行多个操作步骤，并根据页面状态调整策略。

### 3. 数据提取（extract）- 结构化输出

使用 `extract()` 方法从页面提取结构化数据，支持 Zod 类型验证：

```typescript
const { author, title } = await stagehand.extract(
  "extract the author and title of the PR",
  z.object({
    author: z.string().describe("The username of the PR author"),
    title: z.string().describe("The title of the PR"),
  }),
);
```

## 为什么选择 Stagehand？

### 1. 灵活性

- **代码 vs 自然语言**：完全由开发者决定何时使用代码，何时使用自然语言
- **渐进式采用**：可以从简单的代码操作开始，逐步引入 AI 功能
- **场景适配**：适合不同复杂度的自动化需求

### 2. 可维护性

- **预览功能**：在执行 AI 操作前预览，避免误操作
- **缓存机制**：重复操作自动缓存，提升性能并降低成本
- **清晰日志**：详细的操作日志，便于调试和问题追踪

### 3. 可靠性

- **自愈能力**：网站变化时自动检测并智能纠正
- **生产就绪**：设计初衷就是用于生产环境
- **错误处理**：健壮的错误处理和重试机制

### 4. 性能优化

- **CDP 引擎**：基于 Chrome DevTools Protocol，性能优化
- **智能推理**：只在必要时调用 LLM，减少 API 调用
- **批量执行**：支持批量操作，提升效率

## 技术架构

### 核心技术栈

- **TypeScript** - 类型安全的开发语言
- **Playwright** - 底层浏览器自动化框架
- **CDP Protocol** - Chrome DevTools Protocol，提供低级浏览器控制
- **Zod** - TypeScript 优先的模式验证库
- **Browserbase** - 云端浏览器服务

### 支持的浏览器

- **Chromium** 及其衍生浏览器（Chrome, Edge 等）
- **Firefox**
- **WebKit**（Safari）

### 平台支持

- **Node.js** 环境
- **Docker** 容器化部署
- **主流 CI/CD 平台**（GitHub Actions, GitLab CI 等）

## 快速开始

### 安装与初始化

使用一行命令开始：

```bash
npx create-browser-app
```

或从源码构建：

```bash
git clone https://github.com/browserbase/stagehand.git
cd stagehand
pnpm install
pnpm run build
pnpm run example
```

### 环境配置

创建 `.env` 文件并添加必要的 API 密钥：

```bash
cp .env.example .env
nano .env
```

需要配置：
- **LLM Provider API Key**（如 OpenAI, Anthropic 等）
- **Browserbase 凭证**

### Python 版本

如果需要 Python 实现，可以访问：
https://github.com/browserbase/stagehand-python

## 工作流程

### 1. 初始化 Stagehand

```typescript
import { Stagehand } from "@browserbasehq/stagehand";

const stagehand = new Stagehand({
  env: "LOCAL", // 或 "BROWSERBASE"
  modelName: "gpt-4", // LLM 模型
  apiKey: process.env.OPENAI_API_KEY,
});
```

### 2. 导航到页面

```typescript
const page = stagehand.context.pages()[0];
await page.goto("https://example.com");
```

### 3. 执行操作

根据任务复杂度选择合适的方法：

```typescript
// 简单操作
await stagehand.act("click the login button");

// 复杂任务
const agent = stagehand.agent();
await agent.execute("login and navigate to dashboard");

// 数据提取
const data = await stagehand.extract(
  "get product information",
  z.object({
    name: z.string(),
    price: z.number(),
  })
);
```

### 4. 智能缓存

Stagehand 自动缓存已执行的操作：

```typescript
// 第一次执行使用 LLM
await stagehand.act("click the submit button");

// 后续执行直接使用缓存，无需 LLM 调用
await stagehand.act("click the submit button");
```

### 5. 自愈机制

当网站结构发生变化时：

```typescript
let page: Page;
try {
  // 尝试使用缓存的操作
  await stagehand.act("click the new button");
} catch (e) {
  // 自动回退到 AI 推理
  // 并更新缓存
  await stagehand.act("click the new button");
}
```

## 使用场景

### 网站测试自动化

- **端到端测试**：自动化用户流程测试
- **回归测试**：快速验证功能回归
- **兼容性测试**：跨浏览器兼容性验证

### 数据采集

- **网页抓取**：结构化数据提取
- **监控追踪**：价格、库存等实时监控
- **聚合分析**：跨网站数据收集

### 业务流程自动化

- **表单填写**：自动提交各类表单
- **报表生成**：定期生成业务报表
- **客户服务**：自动化客服任务

### CI/CD 集成

- **自动化测试**：集成到 CI/CD 流水线
- **部署验证**：部署后自动验证功能
- **监控警报**：实时监控和报警

## 高级特性

### Director 集成

可以与 Director（https://director.ai）集成，实现更高级的自动化：

```
Vibe code Stagehand with Director 🎭
```

Director 提供可视化的自动化界面和更强大的功能。

### 自定义验证器

使用 Zod 定义复杂的数据验证规则：

```typescript
const schema = z.object({
  products: z.array(
    z.object({
      id: z.string().uuid(),
      name: z.string().min(1),
      price: z.number().positive(),
      inStock: z.boolean(),
    })
  ),
});
```

### 错误处理

完善的错误处理机制：

```typescript
try {
  await stagehand.act("complete purchase");
} catch (error) {
  if (error instanceof StagehandError) {
    // 处理 Stagehand 特定错误
    console.error(error.message);
  } else {
    // 处理其他错误
    console.error("Unexpected error:", error);
  }
}
```

## 性能优化

### 缓存策略

- **操作缓存**：缓存执行成功的操作
- **元素缓存**：缓存页面元素定位器
- **会话缓存**：跨会话保持缓存（可选）

### 智能推理

- **条件触发**：只在必要时调用 LLM
- **批量推理**：多个操作合并推理
- **上下文复用**：复用之前的推理结果

### 并发控制

- **并发限制**：控制并发操作数
- **资源管理**：智能管理浏览器资源
- **内存优化**：优化内存使用

## 社区与支持

### 文档资源

- **官方文档**：https://docs.stagehand.dev
- **快速开始指南**：https://docs.stagehand.dev/v3/first-steps/quickstart
- **示例代码**：项目 `./examples` 目录

### 社区支持

- **Discord 社区**：https://stagehand.dev/discord
- **GitHub Issues**：报告问题和功能请求
- **Twitter/X**：关注项目动态

### 贡献指南

项目高度重视社区贡献，主要关注：

1. **可靠性** - 最优先级
2. **可扩展性**
3. **速度**
4. **成本**

**入门建议**：
- Bug 修复和小改进是最好的切入点
- 大型功能建议先与团队沟通
- 加入 Discord 社区获取指导

## 致谢

项目感谢以下主要贡献者：
- Paul Klein (pkiv)
- Sean McGuire (seanmcguire12)
- Miguel Gonzalez (miguelg719)
- Sameel Arif (sameelarif)
- Thomas Katwan (tkattkat)
- Filip Michalsky (filip-michalsky)
- Anirudh Kamath (kamath)
- Jeremy Press (jeremypress)
- Navid Pour (navidpour)

## 许可证

基于 **MIT License** 开源。

```
Copyright 2025 Browserbase, Inc.

Licensed under the MIT License:
You may use this project for any purpose, with proper attribution.
```

## 相关项目

### 配套工具

- **Stagehand Python**：Python 实现
  https://github.com/browserbase/stagehand-python

- **Browserbase**：云端浏览器服务
  https://www.browserbase.com

- **Director**：可视化自动化界面
  https://director.ai

### 类似工具对比

| 工具 | 优点 | 局限 |
|------|------|------|
| Selenium | 广泛支持 | 代码复杂，维护困难 |
| Playwright | 现代，功能强大 | 需要编写底层代码 |
| Puppeteer | 轻量级 | 仅支持 Chromium |
| Stagehand | AI + 代码，灵活 | 依赖 LLM API |

## 总结

Stagehand 代表了浏览器自动化的新范式，通过巧妙地结合 AI 的智能和代码的精确性，提供了一个既灵活又可靠的自动化解决方案。项目的核心理念是让开发者根据任务复杂度和场景需求，自由选择使用自然语言还是代码，从而实现最佳的开发体验和自动化效果。

在实际应用中，Stagehand 特别适合需要处理动态页面、复杂交互或不确定性的自动化场景。其自愈机制和智能缓存大幅降低了维护成本，使自动化脚本更加健壮和持久。

无论是用于测试自动化、数据采集还是业务流程自动化，Stagehand 都能提供强大的支持。项目的开源特性和活跃的社区也意味着它将持续发展和改进，成为 Web 自动化领域的重要工具。

## 相关资源

- **GitHub 仓库**: https://github.com/browserbase/stagehand
- **官方网站**: https://stagehand.dev
- **完整文档**: https://docs.stagehand.dev
- **Python 版本**: https://github.com/browserbase/stagehand-python
- **许可证**: MIT License
- **技术栈**: TypeScript, Playwright, CDP Protocol, Zod, Browserbase

**标签**: `github` `playwright` `selenium` `puppeteer` `browser-automation` `ai` `LLM` `typescript` `CDP`
