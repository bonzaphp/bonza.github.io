---
layout: post
title: "AI 开发环境依赖安装指南"
date: "2026-03-01 15:13"
categories: ai
tags: [Rust, Go, TypeScript, React, Git, GitHub, API, 数据库]
author: lework
excerpt: 全面的 AI 开发环境搭建指南，涵盖 Rust、Go、TypeScript、React 等技术栈的依赖安装和配置。
---

* content
{:toc}

本文提供了一份全面的 AI 开发环境搭建指南，涵盖了 Rust、Go、TypeScript、React 等主流技术栈的依赖安装和配置。




## Rust 开发环境

### 安装 Rust

```bash
# 使用 rustup 安装 Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# 验证安装
rustc --version
cargo --version
```

### 常用工具

```bash
# 安装 clippy（代码检查）
rustup component add clippy

# 安装 rustfmt（代码格式化）
rustup component add rustfmt

# 安装 cargo-watch（自动重新编译）
cargo install cargo-watch

# 安装 cargo-audit（安全审计）
cargo install cargo-audit
```

## Go 开发环境

### 安装 Go

```bash
# Linux/macOS
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz

# 添加到环境变量
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# 验证安装
go version
```

### Go 工具链

```bash
# 初始化模块
go mod init your-project

# 下载依赖
go mod download

# 整理依赖
go mod tidy

# 安装常用工具
go install golang.org/x/tools/cmd/goimports@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

## Node.js 与 TypeScript

### 安装 Node.js

```bash
# 使用 nvm（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc

# 安装最新 LTS 版本
nvm install --lts
nvm use --lts

# 验证安装
node --version
npm --version
```

### TypeScript 配置

```bash
# 全局安装 TypeScript
npm install -g typescript

# 安装 ts-node
npm install -g ts-node

# 初始化 TypeScript 项目
npm init -y
npm install typescript --save-dev
npx tsc --init
```

## React 开发环境

### Create React App

```bash
# 使用 npm
npx create-react-app my-app
cd my-app
npm start

# 使用 TypeScript 模板
npx create-react-app my-app --template typescript
```

### Vite（更快的构建工具）

```bash
# 创建 Vite + React 项目
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev

# 使用 TypeScript
npm create vite@latest my-app -- --template react-ts
```

## Git 配置

### 基础配置

```bash
# 配置用户信息
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 配置默认分支名
git config --global init.defaultBranch main

# 配置编辑器
git config --global core.editor vim

# 配置凭证存储
git config --global credential.helper store
```

### SSH 密钥

```bash
# 生成 SSH 密钥
ssh-keygen -t ed25519 -C "your.email@example.com"

# 启动 ssh-agent
eval "$(ssh-agent -s)"

# 添加密钥到 ssh-agent
ssh-add ~/.ssh/id_ed25519

# 复制公钥到剪贴板
cat ~/.ssh/id_ed25519.pub | pbcopy  # macOS
# 或
xclip -sel clip < ~/.ssh/id_ed25519.pub  # Linux
```

## 数据库安装

### PostgreSQL

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# macOS (使用 Homebrew)
brew install postgresql
brew services start postgresql

# 创建数据库和用户
sudo -u postgres psql
CREATE DATABASE myapp;
CREATE USER myuser WITH PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE myapp TO myuser;
```

### MongoDB

```bash
# Ubuntu/Debian
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org

# macOS
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

### Redis

```bash
# Ubuntu/Debian
sudo apt install redis-server

# macOS
brew install redis
brew services start redis

# 验证安装
redis-cli ping
```

## API 工具

### curl

```bash
# Ubuntu/Debian
sudo apt install curl

# 验证安装
curl --version
```

### httpie（更友好的 API 客户端）

```bash
# 使用 pip
pip install httpie

# 或使用 apt
sudo apt install httpie

# 使用示例
http GET https://api.github.com/users/octocat
```

### Postman（GUI 工具）

```bash
# 通过 Snap 安装（Linux）
sudo snap install postman

# 或下载 AppImage
wget https://dl.pstmn.io/download/latest/linux64 -O postman-linux-x64.tar.gz
tar -xzf postman-linux-x64.tar.gz
./Postman/Postman
```

## Docker 容器化

### 安装 Docker

```bash
# Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# macOS
brew install --cask docker

# 验证安装
docker --version
docker-compose --version
```

### 常用 Docker 命令

```bash
# 构建镜像
docker build -t my-app .

# 运行容器
docker run -d -p 3000:3000 my-app

# 查看容器
docker ps
docker ps -a

# 停止容器
docker stop container-id

# 删除容器
docker rm container-id

# 查看镜像
docker images

# 删除镜像
docker rmi image-name
```

## 开发工具推荐

### VS Code 扩展

```bash
# 推荐扩展列表
- Rust Analyzer
- Go
- TypeScript and JavaScript Language Features
- ES7+ React/Redux/React-Native snippets
- Prettier - Code formatter
- ESLint
- GitLens
- Docker
-Thunder Client (API 测试)
```

### 终端工具

```bash
# 安装 zsh（更好的 shell）
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 安装 tmux（终端复用）
sudo apt install tmux

# 安装 htop（进程监控）
sudo apt install htop

# 安装 jq（JSON 处理）
sudo apt install jq
```

## 环境验证脚本

创建一个验证脚本来检查所有环境是否正确安装：

```bash
#!/bin/bash
# check-env.sh

echo "检查开发环境..."

# 检查 Rust
if command -v rustc &> /dev/null; then
    echo "✓ Rust: $(rustc --version)"
else
    echo "✗ Rust 未安装"
fi

# 检查 Go
if command -v go &> /dev/null; then
    echo "✓ Go: $(go version)"
else
    echo "✗ Go 未安装"
fi

# 检查 Node.js
if command -v node &> /dev/null; then
    echo "✓ Node.js: $(node --version)"
else
    echo "✗ Node.js 未安装"
fi

# 检查 Git
if command -v git &> /dev/null; then
    echo "✓ Git: $(git --version)"
else
    echo "✗ Git 未安装"
fi

# 检查 Docker
if command -v docker &> /dev/null; then
    echo "✓ Docker: $(docker --version)"
else
    echo "✗ Docker 未安装"
fi

echo "检查完成！"
```

## 总结

通过本指南，你应该能够成功搭建一个完整的 AI 开发环境。记住：

1. 定期更新所有工具和依赖
2. 使用版本管理器（如 nvm、rustup）以便轻松切换版本
3. 配置好代码编辑器和开发工具以提高效率
4. 保持环境干净，及时清理不需要的依赖

---

*本文基于常见开发环境配置整理，内容来源于官方文档和社区最佳实践。*