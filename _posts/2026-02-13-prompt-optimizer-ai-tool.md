---
layout: post
title:  "Prompt Optimizer：强大的AI提示词优化工具，助力编写高质量提示词"
date:   2026-02-13 00:21:00 +0800
categories: ai
tags: ai-tools prompt-engineering prompt-optimizer github 提示词优化
author: 来财
---


* content
{:toc}




## 🚀 Prompt Optimizer：智能提示词优化器

Prompt Optimizer 是一个强大的 AI 提示词优化工具，专门帮助用户编写更好的 AI 提示词，显著提升 AI 输出质量。该工具支持多种部署方式，包括 Web 应用、桌面应用、Chrome 插件和 Docker 部署，为不同使用场景提供灵活的解决方案。

## 📖 项目简介

### 🎯 核心定位
Prompt Optimizer 致力于解决 AI 提示词编写中的核心痛点：如何让 AI 理解用户的真实意图并产生高质量的输出。通过智能优化算法和多种测试模式，该工具能够将模糊的想法转化为精准的提示词。

### 🎥 功能演示

#### 1. 角色扮演对话：激发小模型潜力
在追求成本效益的生产或注重隐私的本地化场景中，结构化的提示词能让小模型稳定地进入角色，提供沉浸式、高一致性的角色扮演体验，有效激发其潜力。

#### 2. 知识图谱提取：保障生产环境稳定性
在需要程序化处理的生产环境中，高质量的提示词能显著降低对模型智能程度的要求，使得更经济的小模型也能稳定输出可靠的指定格式。本工具旨在辅助开发者快速达到此目的，从而加速开发、保障稳定，实现降本增效。

#### 3. 诗歌写作：辅助创意探索与需求定制
当面对一个强大的 AI，我们的目标不只是得到一个"好"答案，而是得到一个"我们想要的"独特答案。本工具能帮助用户将一个模糊的灵感（如"写首诗"）细化为具体的需求（关于什么主题、何种意象、何种情感），辅助您探索、发掘并精确表达自己的创意，与 AI 共创独一无二的作品。

## ✨ 核心特性

### 🎯 智能优化
- **一键优化**：支持一键优化提示词，提升 AI 回复准确度
- **多轮迭代**：支持多轮迭代改进，持续优化提示词质量
- **双模式优化**：支持系统提示词优化和用户提示词优化
- **对比测试**：支持原始提示词和优化后提示词的实时对比

### 🤖 多模型集成
- **主流 AI 模型**：支持 OpenAI、Gemini、DeepSeek、智谱AI、SiliconFlow 等
- **自定义 API**：支持 OpenAI 兼容接口的自定义模型
- **无限扩展**：支持配置无限数量的自定义模型
- **参数调优**：支持各模型特有参数配置

### 🖼️ 图像生成
- **文生图（T2I）**：通过文本提示词生成图像
- **图生图（I2I）**：基于本地图片进行图像变换和优化
- **多模型支持**：集成 Gemini、Seedream 等主流图像生成模型
- **参数配置**：支持各模型特有参数配置（如尺寸、风格等）

### 📊 高级测试模式
- **上下文变量管理**：自定义变量、批量替换、变量预览
- **多轮会话测试**：模拟真实对话场景，测试提示词在多轮交互中的表现
- **工具调用支持**：Function Calling 集成，支持 OpenAI 和 Gemini 工具调用
- **灵活调试**：更强大的提示词测试和调试能力

### 🔒 安全架构
- **纯客户端处理**：数据直接与 AI 服务商交互，不经过中间服务器
- **本地存储**：所有数据只存储在浏览器本地，不会上传至任何服务器
- **访问控制**：支持密码保护功能，保障部署安全
- **跨域解决方案**：桌面应用彻底解决跨域问题

### 📱 多端支持
- **Web 应用**：基于浏览器的在线版本
- **桌面应用**：支持 Windows、macOS、Linux 的原生应用
- **Chrome 插件**：浏览器插件形式，方便快速使用
- **Docker 部署**：容器化部署，适合生产环境

### 🧩 MCP 协议支持
- **MCP Server**：支持 Model Context Protocol (MCP) 协议
- **Claude Desktop 集成**：可与 Claude Desktop 等 MCP 兼容应用集成
- **标准化接口**：提供标准化的工具接口
- **扩展生态**：融入 MCP 工具生态

## 🚀 快速开始

### 1. 使用在线版本（推荐）
直接访问：[https://prompt.always200.com](https://prompt.always200.com)

项目是纯前端项目，所有数据只存储在浏览器本地，不会上传至任何服务器，因此直接使用在线版本也是安全可靠的。

### 2. Vercel 部署

#### 方式1：一键部署
https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Flinshenkx%2Fprompt-optimizer

#### 方式2：Fork 后部署（推荐）
- 先 Fork 项目到自己的 GitHub
- 然后在 Vercel 中导入该项目
- 配置环境变量：
  - `ACCESS_PASSWORD`：设置访问密码，启用访问限制
  - `VITE_OPENAI_API_KEY` 等：配置各 AI 服务商的 API 密钥

### 3. 下载桌面应用
从 [GitHub Releases](https://github.com/linshenkx/prompt-optimizer/releases) 下载最新版本。

#### 桌面应用核心优势
- ✅ **无跨域限制**：作为原生桌面应用，能彻底摆脱浏览器跨域（CORS）问题的困扰
- ✅ **自动更新**：通过安装程序安装的版本，能够自动检查并更新到最新版
- ✅ **独立运行**：无需依赖浏览器，提供更快的响应和更佳的性能

### 4. 安装 Chrome 插件
从 Chrome 商店安装：[Chrome 商店地址](https://chromewebstore.google.com/detail/prompt-optimizer/cakkkhboolfnadechdlgdcnjammejlna)

### 5. Docker 部署
```bash
# 运行容器（默认配置）
docker run -d -p 8081:80 --restart unless-stopped --name prompt-optimizer linshen/prompt-optimizer

# 运行容器（配置 API 密钥和访问密码）
docker run -d -p 8081:80 \
  -e VITE_OPENAI_API_KEY=your_key \
  -e ACCESS_USERNAME=your_username \
  -e ACCESS_PASSWORD=your_password \
  --restart unless-stopped \
  --name prompt-optimizer \
  linshen/prompt-optimizer
```

### 6. Docker Compose 部署
```yaml
# docker-compose.yml
services:
  prompt-optimizer:
    image: linshen/prompt-optimizer:latest
    container_name: prompt-optimizer
    restart: unless-stopped
    ports:
      - "8081:80"
    environment:
      - VITE_OPENAI_API_KEY=your_openai_key
      - VITE_GEMINI_API_KEY=your_gemini_key
      - ACCESS_USERNAME=admin
      - ACCESS_PASSWORD=your_password
```

## ⚙️ API 密钥配置

### 方式一：通过界面配置（推荐）
1. 点击界面右上角的"⚙️设置"按钮
2. 选择"模型管理"选项卡
3. 点击需要配置的模型（如 OpenAI、Gemini、DeepSeek 等）
4. 在弹出的配置框中输入对应的 API 密钥
5. 点击"保存"即可

### 方式二：通过环境变量配置
Docker 部署时通过 `-e` 参数配置环境变量：
```bash
-e VITE_OPENAI_API_KEY=your_key
-e VITE_GEMINI_API_KEY=your_key
-e VITE_DEEPSEEK_API_KEY=your_key
-e VITE_ZHIPU_API_KEY=your_key
-e VITE_SILICONFLOW_API_KEY=your_key

# 多自定义模型配置（支持无限数量）
-e VITE_CUSTOM_API_KEY_ollama=dummy_key
-e VITE_CUSTOM_API_BASE_URL_ollama=http://localhost:11434/v1
-e VITE_CUSTOM_API_MODEL_ollama=qwen2.5:7b
```

### 高级 LLM 参数配置
除了 API 密钥，还可以为每个模型单独设置高级 LLM 参数：

#### OpenAI/兼容 API
```json
{
  "temperature": 0.7,
  "max_tokens": 4096,
  "timeout": 60000
}
```

#### Gemini
```json
{
  "temperature": 0.8,
  "maxOutputTokens": 2048,
  "topP": 0.95
}
```

#### DeepSeek
```json
{
  "temperature": 0.5,
  "top_p": 0.9,
  "frequency_penalty": 0.1
}
```

## 🧩 MCP Server 使用说明

### 环境变量配置
```bash
# MCP 服务器配置
MCP_DEFAULT_MODEL_PROVIDER=openai  # 可选值：openai, gemini, anthropic, deepseek, siliconflow, zhipu, dashscope, openrouter, modelscope, custom
MCP_LOG_LEVEL=info  # 日志级别
```

### Claude Desktop 集成示例
编辑 Claude Desktop 的配置文件（`services.json`）：
```json
{
  "services": [
    {
      "name": "Prompt Optimizer",
      "url": "http://localhost:8081/mcp"
    }
  ]
}
```

### 可用工具
- **optimize-user-prompt**：优化用户提示词以提高 LLM 性能
- **optimize-system-prompt**：优化系统提示词以提高 LLM 性能
- **iterate-prompt**：对已经成熟/完善的提示词进行定向迭代优化

## 🗺️ 开发路线

### 已完成功能
- ✅ 基础功能开发
- ✅ Web 应用发布
- ✅ Chrome 插件发布
- ✅ 国际化支持
- ✅ 系统提示词优化和用户提示词优化
- ✅ 桌面应用发布
- ✅ MCP 服务发布
- ✅ 高级模式：变量管理、上下文测试、工具调用
- ✅ 图像生成：文生图（T2I）和图生图（I2I）支持

### 计划中功能
- 🔄 支持工作区/项目管理
- 🔄 支持提示词收藏和模板管理

## ❓ 常见问题

### API 连接问题

#### Q1: 为什么配置好 API 密钥后仍然无法连接到模型服务？
A: 大多数连接失败是由跨域问题（CORS）导致的。由于本项目是纯前端应用，浏览器出于安全考虑会阻止直接访问不同源的 API 服务。

#### Q2: 如何解决本地 Ollama 的连接问题？
A: Ollama 完全支持 OpenAI 标准接口，只需配置正确的跨域策略：
- 设置环境变量 `OLLAMA_ORIGINS=*` 允许任意来源的请求
- 如仍有问题，设置 `OLLAMA_HOST=0.0.0.0:11434` 监听任意 IP 地址

#### Q3: 如何解决商业 API 的跨域问题？
A: 推荐以下解决方案：
- **使用桌面版应用**（最推荐）：完全没有跨域限制
- **使用自部署的 API 中转服务**：部署如 OneAPI、NewAPI 等开源 API 聚合/代理工具

#### Q4: 为什么使用在线版无法连接本地模型？
A: 这是由浏览器的混合内容（Mixed Content）安全策略导致的。浏览器会阻止安全的 HTTPS 页面向不安全的 HTTP 地址发送请求。

解决方案：
- 使用桌面版：最稳定可靠的方式
- 使用 Docker 部署（HTTP）：通过 http://localhost:8081 访问
- 使用 Chrome 插件：可以绕过部分安全限制

### macOS 桌面应用问题

#### Q5: macOS 打开应用时提示「已损坏」或「无法验证开发者」怎么办？
A: 在终端中执行以下命令移除安全隔离属性：
```bash
# 对于已安装的应用
xattr -rd com.apple.quarantine /Applications/PromptOptimizer.app

# 对于下载的 .dmg 文件（安装前执行）
xattr -rd com.apple.quarantine ~/Downloads/PromptOptimizer-*.dmg
```

## 🤝 参与贡献

### 贡献流程
1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m '添加某个特性'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

### 代码审查建议
使用 cursor 工具开发时，建议在提交前：
- 使用"code_review"规则进行代码审查
- 检查变更的整体一致性
- 评估代码质量和实现方式
- 确认测试覆盖情况
- 完善文档程度

## 📄 开源协议

本项目采用 [AGPL-3.0](https://github.com/linshenkx/prompt-optimizer/blob/develop/LICENSE) 协议开源。

### 允许做什么
- ✅ 个人使用、学习、研究
- ✅ 公司内部使用（不对外提供服务）
- ✅ 修改代码并用于商业项目
- ✅ 收费销售或提供服务

### 需要做什么
- 📖 如果分发软件或提供网络服务，必须公开源代码
- 📝 保留原作者的版权声明

**一句话核心：可以商用，但不能闭源。**

## 🎯 应用场景

### 👨‍💻 个人开发者
- **提示词学习**：学习如何编写高质量的提示词
- **AI 对话优化**：改善与 AI 的对话体验
- **创意写作**：辅助创作诗歌、故事等文学作品
- **技术问题解决**：优化技术问题的描述，获得更好的解决方案

### 👥 团队协作
- **标准化提示词**：建立团队统一的提示词标准
- **知识管理**：将团队经验转化为可复用的提示词模板
- **培训支持**：帮助团队成员提升 AI 使用技能
- **效率提升**：减少无效的 AI 对话，提高工作效率

### 🏢 企业应用
- **客户服务**：优化客服机器人的回复质量
- **内容生成**：提高营销内容、产品描述的生成质量
- **数据分析**：优化数据分析提示词，获得更准确的洞察
- **产品开发**：辅助产品需求分析和功能设计

## 🚀 技术优势

### 🏗️ 架构设计
- **纯前端架构**：无服务器依赖，降低部署复杂度
- **模块化设计**：核心功能模块化，易于扩展和维护
- **跨平台支持**：支持多种部署方式，适应不同环境
- **标准化接口**：采用 MCP 等标准协议，便于集成

### 🔒 安全特性
- **数据隐私**：所有数据处理都在本地进行
- **访问控制**：支持密码保护和访问限制
- **安全传输**：直接与 AI 服务商建立安全连接
- **无中间服务器**：避免数据泄露风险

### ⚡ 性能优化
- **本地缓存**：智能缓存优化响应速度
- **异步处理**：异步 API 调用提升用户体验
- **资源优化**：优化资源加载和内存使用
- **批量操作**：支持批量优化和处理

## 📈 项目影响

### 🌟 社区反响
- **GitHub Stars**：项目获得广泛关注和认可
- **用户反馈**：积极的用户评价和使用反馈
- **持续更新**：活跃的开发和功能迭代
- **文档完善**：详细的用户文档和开发指南

### 🎯 技术创新
- **提示词工程**：推动提示词工程标准化
- **工具集成**：集成多种 AI 工具和协议
- **用户体验**：优化 AI 工具使用体验
- **开源贡献**：为开源社区贡献有价值工具

## 🎉 总结

Prompt Optimizer 通过其强大的提示词优化能力和多样化的部署方式，为 AI 用户提供了完整的提示词工程解决方案。它不仅解决了提示词编写的技术难题，还通过丰富的功能和良好的用户体验，大大提升了 AI 工具的使用效率。

### 🎯 核心价值

1. **质量提升**：显著提高 AI 输出质量
2. **效率优化**：减少无效对话，提高工作效率
3. **成本控制**：通过优化提示词降低 API 使用成本
4. **标准化**：建立提示词工程的最佳实践
5. **易用性**：提供简单易用的界面和操作

### 🚀 技术优势

- **多端支持**：适应不同使用场景和环境
- **安全可靠**：保障数据隐私和安全
- **标准协议**：支持 MCP 等行业标准
- **持续演进**：活跃的开发和功能迭代

Prompt Optimizer 代表了 AI 提示词工程工具的重要发展方向，为用户提供了专业、高效、安全的提示词优化解决方案，是 AI 时代不可或缺的生产力工具。

---

*本文基于 Prompt Optimizer 项目的官方 README 文件整理，更多详细信息请参考项目 GitHub 仓库。*