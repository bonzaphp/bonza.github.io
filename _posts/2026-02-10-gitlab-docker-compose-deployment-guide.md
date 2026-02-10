---
layout: post
title:  "GitLab Docker Compose 部署指南：集成 Jenkins 的完整解决方案"
date:   2026-02-10 20:01:00 +0800
categories: devops
tags: gitlab docker compose jenkins devops 容器化部署
author: 来财
---


* content
{:toc




本文详细介绍了使用 Docker Compose 部署 GitLab 和 Jenkins 的完整方案，包括镜像选择、配置文件详解、网络配置、数据持久化、邮件服务配置、LDAP 集成以及常见问题的解决方案，帮助快速搭建完整的 DevOps 工作流。

# GitLab Docker Compose 部署指南：集成 Jenkins 的完整解决方案

## 前言

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。通过使用 Docker Compose 部署 GitLab 和 Jenkins，我们可以快速搭建一个完整的 DevOps 工作流，实现代码托管、CI/CD、自动化测试和部署的一体化解决方案。

本文将详细介绍如何使用 Docker Compose 部署 GitLab CE 中文版，并集成 Jenkins 实现自动化流水线。

## 一、架构概览

### 1. 系统架构

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        Internet                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Load Balancer (Nginx)                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                GitLab Container (Port 80, 443, 22)                │   │
│  │  ┌───────────────────────────────────────────────────────────────┐   │
│  │  │              GitLab Application                              │   │
│  │  │              - Web Interface                           │   │
│  │  │              - Git Repository                        │   │
│  │  │              - CI/CD Pipelines                        │   │
│  │  │              - Container Registry                    │   │
│  │  └───────────────────────────────────────────────────────────────┘�   │
│  └─────────────────────────────────────────────────────────────────────┘�   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                Jenkins Container (Port 9080, 50000)                │   │
│  │  ┌───────────────────────────────────────────────────────────────┐   │
│  │  │              Jenkins Server                             │   │
│  │  │              - Build Automation                         │   │
│  │  │              - Pipeline Management                    │   │
│  │  │              - Artifact Repository                    │   │
│  │  │              - Plugin Ecosystem                       │   │
│  │  └───────────────────────────────────────────────────────────────┘�   │
│  └─────────────────────────────────────────────────────────────────────┘�   │
└─────────────────────────────────────────────────────────────────────────────────┘�
```

### 2. 组件说明

#### GitLab 组件
- **GitLab CE**: 中文版 GitLab 社区版
- **PostgreSQL**: 数据库服务
- **Redis**: 缓存服务
- **Nginx**: Web 服务器
- **Sidekiq**: 后台作业处理器
- **Unicorn**: 应用服务器

#### Jenkins 组件
- **Jenkins Server**: Jenkins 主服务
- **Build Agent**: 构建代理
- **Plugin Ecosystem**: 插件系统

## 二、Docker Compose 配置文件详解

### 1. 完整的 docker-compose.yml

```yaml
version: '2'

services:
  gitlab:
    image: 'twang2218/gitlab-ce-zh:11.1.4'
    container_name: gitlab
    restart: unless-stopped
    hostname: 'gitlab.example.com'
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
        sidekiq['concurrency'] = 5
        unicorn['worker_processes'] = 2
        
        # SMTP 邮件服务配置
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = "smtp.mxhichina.com"
        gitlab_rails['smtp_port'] = 465
        gitlab_rails['smtp_user_name'] = "admin@bonza.cn"
        gitlab_rails['smtp_password'] = "qwe123"
        gitlab_rails['smtp_domain'] = "smtp.mxhichina.com"
        gitlab_rails['smtp_authentication'] = "login"
        gitlab_rails['smtp_enable_starttls_auto'] = true
        gitlab_rails['smtp_tls'] = true
        
        # 邮件服务配置
        gitlab_rails['gitlab_email_enabled'] = true
        gitlab_rails['gitlab_email_from'] = 'admin@bonza.cn'
        gitlab_rails['gitlab_email_display_name'] = 'GitLab'
        
        # LDAP 认证配置
        gitlab_rails['ldap_enabled'] = true
        gitlab_rails['ldap_servers'] = {
          'main' => {
            'label' => 'LDAP',
            'host' => 'ldap.mydomain.com',
            'port' => 636,
            'uid' => 'sAMAccountName',
            'encryption' => 'simple_tls',
            'base' => 'dc=example,dc=com',
          }
        }
    ports:
      - '7780:80'      # HTTP
      - '443:443'      # HTTPS
      - '22:22'        # SSH
    volumes:
      - config:/etc/gitlab
      - data:/var/opt/gitlab
      - logs:/var/log/gitlab
    networks:
      - gitlab-jenkins-net

  jenkins:
    image: 'jenkinsci/blueocean:1.25.6'
    container_name: jenkinsci
    privileged: true
    restart: unless-stopped
    hostname: 'jenkins.example.com'
    environment:
      JAVA_OPTS: "-Duser.timezone=Asia/Shanghai"
      # 关闭跨站攻击防护（可选）
      # JAVA_OPTS: "-Duser.timezone=Asia/Shanghai,-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true"
    ports:
      - '9080:8080'    # Jenkins Web 界面
      - '51000:50000'  # JNLP 端口
    extra_hosts:
      - "get.jenkins.io:52.167.253.43"
    volumes:
      - jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /etc/sysconfig/docker:/etc/sysconfig/docker
      - /etc/localtime:/etc/localtime:ro
    networks:
      - gitlab-jenkins-net

volumes:
  config:
    driver: local
    driver_opts:
      type: bind
      o: bind
      device: ${PWD}/gitlab/config
  data:
    driver: local
    driver_opts:
      type: bind
      o: bind
      device: ${PWD}/gitlab/data
  logs:
    driver: local
    driver_opts:
      type: bind
      o: bind
      device: ${PWD}/gitlab/logs
  jenkins:
    driver: local
    driver_opts:
      type: bind
      o: bind
      device: ${PWD}/jenkinsci/data

networks:
  gitlab-jenkins-net:
    driver: bridge
```

### 2. 配置文件详细解析

#### GitLab 配置项

```yaml
# GitLab 镜像选择
image: 'twang2218/gitlab-ce-zh:11.1.4'  # 中文版 GitLab CE
# image: 'gitlab/gitlab-ce:15.9.3-ce.0'   # 官�方版本

# 容器管理
container_name: gitlab
restart: unless-stopped
hostname: 'gitlab.example.com'

# 环境变量配置
environment:
  TZ: 'Asia/Shanghai'  # 时区设置
  
  # GitLab Omnibus 配置
  GITLAB_OMNIBUS_CONFIG: |
    external_url 'http://gitlab.example.com'
    gitlab_rails['time_zone'] = 'Asia/Shanghai'
    sidekiq['concurrency'] = 5
    unicorn['worker_processes'] = 2
```

#### SMTP 邮件服务配置

```yaml
# SMTP 基本配置
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.mxhichina.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "admin@bonza.cn"
gitlab_rails['smtp_password'] = "qwe123"
gitlab_rails['smtp_domain'] = "smtp.mxhichina.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

# 邮件显示配置
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'admin@bonza.cn'
gitlab_rails['gitlab_email_display_name'] = 'GitLab'
```

#### LDAP 认证配置

```yaml
# LDAP 启用
gitlab_rails['ldap_enabled'] = true

# LDAP 服务器配置
gitlab_rails['ldap_servers'] = {
  'main' => {
    'label' => 'LDAP',
    'host' => 'ldap.mydomain.com',
    'port' => 636,
    'uid' => 'sAMAccountName',
    'encryption' => 'simple_tls',
    'base' => 'dc=example,dc=com',
  }
}
```

#### Jenkins 配置项

```yaml
# Jenkins 镜像选择
image: 'jenkinsci/blueocean:1.25.6'

# 权限配置
privileged: true  # 需要 Docker socket 访问权限

# Java 环境变量
environment:
  JAVA_OPTS: "-Duser.timezone=Asia/Shanghai"
  
# 安全配置（可选）
# JAVA_OPTS: "-Duser.timezone=Asia/Shanghai,-Dhudson.security.csrf.GlobalCruminIssuerConfiguration.DISABLE_CSRF_PROTECTION=true"

# 主机名配置
hostname: 'jenkins.example.com'

# 额外主机解析
extra_hosts:
  - "get.jenkins.io:52.167.253.43"
```

## 三、部署前准备

### 1. 系统要求

#### 硬件配置
```bash
# 检查系统版本
cat /etc/os-release

# 检查 Docker 版本
docker --version
docker-compose --version
```

#### 硬件要求
- **操作系统**: Ubuntu 18.04+ / CentOS 7+ / RHEL 7+
- **Docker**: 20.10.0+
- **Docker Compose**: 2.0.0+
- **内存**: 至少 4GB RAM
- **存储**: 至少 50GB 可用空间

### 2. 安装 Docker 和 Docker Compose

#### Ubuntu/Debian 系统

```bash
# 更新包索引
apt update

# 安装依赖
apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg

# 添加 Docker 官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker 仓库
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "UBUNTU_CODENAME=$(grep VERSION_CODENAME /etc/os-release | cut -d=2 -f2 | tr '[:upper:]' '[:lower:]')"
case "$UBUNTU_CODENAME" in
  focal)
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu focal"
    ;;
esac
    ;;
esac | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动 Docker 服务
systemctl start docker
systemctl enable docker
```

#### CentOS/RHEL 系统

```bash
# 安装依赖
yum install -y yum-utils

# 添加 Docker 仓库
yum-config-manager --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker
yum install -y docker-ce docker-ce-cli docker-compose-plugin

# 启动 Docker 服务
systemctl start docker
systemctl enable docker
```

### 3. 创建目录结构

```bash
# 创建项目目录
mkdir -p gitlab/{config,data,logs}
mkdir -p jenkins/data

# 设置权限
chmod -R 755 gitlab/
chmod -R 755 jenkins/
chown -R $(id -u):$(id -g) gitlab/
chown -R $(id -u):$(id -g) jenkins/

# 验证目录结构
tree -L gitlab/ jenkins/
```

## 四、部署操作

### 1. 创建配置文件

#### 创建 docker-compose.yml

```bash
# 创建项目目录
mkdir -p gitlab-docker-compose
cd gitlab-docker-compose

# 创建 docker-compose.yml 文件
cat > docker-compose.yml << 'EOF'
version: '2'

services:
  gitlab:
    image: 'twang2218/gitlab-ce-zh:11.1.4'
    container_name: gitlab
    restart: unless-stopped
    hostname: 'gitlab.example.com'
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
        sidekiq['concurrency'] = 5
        unicorn['worker_processes'] = 2
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = "smtp.mxhichina.com"
        gitlab_rails['smtp_port'] = 465
        gitlab_rails['smtp_user_name'] = "admin@bonza.cn"
        gitlab GitLab_rails['smtp_password'] = "qwe123"
        gitlab_rails['smtp_domain'] = "smtp.mxhichina.com"
        gitlab_rails['smtp_authentication'] = "login"
        gitlab_rails['smtp_enable_starttls_auto'] = true
        gitlab_rails['smtp_tls'] = true
        gitlab_rails['gitlab_email_enabled'] = true
        gitlab_rails['gitlab_email_from'] = 'admin@bonza.cn'
        gitlab_rails['gitlab_email_display_name'] = 'GitLab'
        gitlab_rails['ldap_enabled'] = true
        gitlab_rails['ldap_servers'] = {
          'main' => {
            'label' => 'LDAP',
            'host' => 'ldap.mydomain.com',
            'port' => 636,
            'uid' => 'sAMAccountName',
            'encryption' => 'simple_tls',
            'base' => 'dc=example,dc=com',
          }
        }
    ports:
      - '7780:80'
      - '443:443'
      - '22:22'
    volumes:
      - config:/etc/gitlab
      - data:/var/opt/gitlab
      - logs:/var/log/gitlab
    networks:
      - gitlab-jenkins-net

  jenkins:
    image: 'jenkinsci/blueocean:1.25.6'
    container_name: jenkinsci
    privileged: true
    restart: unless-stopped
    hostname: 'gitlab.example.com'
    environment:
      JAVA_OPTS: "-Duser.timezone=Asia/Shanghai"
    ports:
      - '9080:8080'
      - '51000:50000'
    extra_hosts:
      - "get.jenkins.io:52.167.253.43"
    volumes:
      - jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /etc/sysconfig/docker:/etc/sysconfig/docker
      - /etc/localtime:/etc/localtime:ro
    networks:
      - gitlab-jenkins-net

volumes:
  config:
    driver: local
    driver_opts:
      type: bind
      o: bind
      device: ${PWD}/gitlab/config
  data:
    driver: local
    driver_opts:
      type: bind
      o: bind
      domain: device: ${PWD}/gitlab/data
  logs:
    driver: local
    driver_opts:
      pull: bind
      o: bind
      device: ${PWD}/gitlab/logs
  jenkins:
    driver: local
    driver_opts:
      type: bind
      o: bind
      device: ${PWD}/jenkinsci/data

networks:
  gitlab-jenkins-net:
    driver: bridge
EOF
```

### 2. 启动服务

#### 启动 Docker Compose 服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看服务日志
docker-compose logs -f
```

#### 启动特定服务

```bash
# 只启动 GitLab
docker-compose up -d gitlab

# 只启动 Jenkins
docker-compose up -d jenkins

# 重启服务
docker-compose restart gitlab
docker-compose restart jenkins
```

### 3. 验证部署

#### 检查服务状态

```bash
# 检查所有服务状态
docker-compose ps

# 检查容器健康状态
docker-compose exec gitlab gitlab-ctl status

# 检查 Jenkins 状态
docker-compose exec jenkins curl -s http://localhost:9080
```

#### 访问 Web 界面

```bash
# GitLab 访问地址
http://gitlab.example.com

# Jenkins 访问地址
http://jenkins.example.com:9080

# SSH 访问 GitLab
ssh git clone git@gitlab.example.com:group/project.git
```

## 五、配置详解

### 1. GitLab 高级配置

#### 外部 URL 配置

```yaml
# 在 docker-compose.yml 的 environment 部分
external_url 'https://gitlab.example.com'
gitlab_rails['gitlab_https'] = true
gitlab_rails['gitlab_port'] = 443
gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
gitlab_rails['gitlab_shell_ssh_port'] = 2222
```

#### 数据库配置

```yaml
# PostgreSQL 配置
postgresql['shared_preload_libraries'] = false
postgresql['data_dir'] = '/var/opt/gitlab/postgresql/data'
postgresql['listen_address'] = '0.0.0.0'
postgresql['port'] = 5432
postgresql['sql_user'] = 'gitlab'
postgresql['group'] = 'gitlab'
```

#### Redis 配置

```yaml
# Redis 配置
redis['bind'] = '127.0.0.1'
redis['port'] = 6379
redis['timeout'] = 5
redis['save'] = 900 1 300 10 60
redis['maxmemory'] = '2gb'
redis['maxmemory-policy'] = 'allkeys-lru'
```

### 2. Jenkins 配置优化

#### JVM 内存配置

```yaml
# 在 docker-compose.yml 中调整 JVM 参数
environment:
  JAVA_OPTS: >
    -Duser.timezone=Asia/Shanghai
    -Xms512m
    -Xmx2048m
    -XX:MaxMetaspaceSize=512m
    -XX:+UseCompressedOops
```

#### 插件配置

```bash
# 进入 Jenkins 容器
docker-compose exec jenkins bash

# 安装插件
jenkins-plugin-cli --list
jenkins-plugin-cli install gitlab
jenkins-plugin-cli install git
jenkins-plugin locale
jenkins-plugin-cli install blueocean
```

#### 系统配置

```bash
# 设置 Jenkins 时区
docker-compose exec jenkins timedatectl set-timezone Asia/Shanghai

# 配置 Git
docker-compose exec jenkins git config --global user.name "Jenkins Admin"
docker-compose exec jenkins git config --global user.email "admin@example.com"
```

## 六、网络配置

### 1. 网络架构

#### 网络拓扑

```
Internet
    │
    └─ nginx (80/443)
        │
        └─ gitlab (7780/443) ──── jenkins (9080)
        │           │
        │           └─ gitlab-jenkins-net
        │               │
        │               └─ 通信
```

#### 网络配置文件

```yaml
networks:
  gitlab-jenkins-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
        - gateway: 172.20.0.1
        - ip_range: 172.20.0.2-172.20.0.254
```

### 2. 端口映射

#### 端口配置说明

| 服务 | 端口 | 说明 |
|------|------|------|
| GitLab HTTP | 7780:80 | GitLab Web 界面 |
| GitLab HTTPS | 443:443 | GitLab 安全访问 |
| GitLab SSH | 22:22 | Git Shell 访问 |
| Jenkins HTTP | 9080:8080 | Jenkins Web 端面 |
| Jenkins JNLP | 51000:50000 | Jenkins JNLP 端口 |

#### 反向代理配置（可选）

```yaml
# nginx.conf 示例
server {
    listen 80;
    server_name gitlab.example.com;
    
    location / {
        proxy_pass http://gitlab-jenkins-net:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /jenkins {
        proxy_pass http://gitlab-jenkins-net:9080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 七、数据持久化

### 1. 数据卷配置

#### 重要数据目录

```bash
# GitLab 数据卷
volumes:
  - config:/etc/gitlab          # GitLab 配置文件
  - data:/var/opt/gitlab        # 应用数据
  - logs:/var/log/gitlab          # 日志文件
```

#### Jenkins 数据卷

```bash
# Jenkins 数据卷
volumes:
  - jenkins:/var/jenkins_home    # Jenkins 主目录
  - /var/run/docker.sock:/var/run/docker.sock  # Docker socket
  - /usr/bin/docker:/usr/bin/docker      # Docker CLI
  - /etc/localtime:/etc/localtime:ro  # 时区信息
```

### 2. 数据备份策略

#### 备份 GitLab 数据

```bash
#!/bin/bash
# GitLab 数据备份脚本

BACKUP_DIR="/backup/gitlab"
DATE=$(date +%Y%m%d)

# 备份数据卷
docker run --rm -v gitlab_config_backup gitlab_config
docker run --v gitlab_data_backup gitlab_data
docker run --v gitlab_logs_backup gitlab_logs

# 备份文件
docker run --rm -v gitlab_config_backup \
  -v $(docker volume inspect gitlab_config --format '{{ .Mountpoint }}'):/backup/gitlab_config
docker run --rm -v gitlab_data_backup \
  -v $(docker volume inspect gitlab_data --format '{{ .Mountpoint }}'):/backup/gitlab_data
docker run --rm -v gitlab_logs_backup \
  -v $(docker inspect gitlab_logs --format '{{ .Mountpoint }}'):/backup/gitlab_logs

# 压缩备份
tar -czf "gitlab_backup_$DATE.tar.gz" /backup/gitlab/
```

#### 恢复 GitLab 数据

```bash
#!/bin/bash
# GitLab 数据恢复脚本

BACKUP_FILE="gitlab_backup_20231210.tar.gz"

# 停止服务
docker-compose down

# 恢复数据卷
docker run --rm gitlab_config_restore \
  -v /backup/gitlab_config:/etc/gitlab
docker run --rm gitlab_data_restore \
  -v /backup/gitlab_data:/var/opt/gitlab
docker run --rm gitlab_logs_restore \
  - /backup/gitlab_logs:/var/log/gitlab

# 重启服务
docker-compose up -d
```

## 八、集成配置

### 1. GitLab CI/CD 集成 Jenkins

#### GitLab Runner 配置

```yaml
# gitlab-ci.yml 示例
image: jenkins/jenkins:lts
services:
  - name: docker:dind
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

variables:
  DOCKER_HOST: tcp://docker:2375
  DOCKER_TLS_CERTDIR: /certs/client
  DOCKER_TLS_VERIFY: 0

stages:
  - build
  - test
  - deploy
```

#### Jenkins Pipeline 配置

```groovy
// Jenkinsfile
pipeline {
    agent {
        docker {
            image 'jenkins/jenkins:lts'
        }
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    echo 'Building application...'
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying to production...'
            }
        }
}
```

### 2. Webhook 配置

#### GitLab Webhook 配置

```bash
# 在 GitLab 中配置 Jenkins 集成
# 1. 项目 → Settings → Integrations → Jenkins
# 2. 设置 URL: http://jenkins.example.com:9080
# 3. 添加 Webhook URL
# 4. 设置触发条件
```

#### Jenkins Webhook 接收

```bash
# Jenkins Webhook 接收配置
# Jenkins → 项目 → Configure → Webhook
# 1. 选择 "GitLab"
# 2. 输入 GitLab 服务器 URL
# 3. 添加凭据
# 4. 测试连接
```

## 九、监控和日志

### 1. 日志管理

#### 查看 GitLab 日志

```bash
# 查看所有服务日志
docker-compose logs

# 查看 GitLab 应用日志
docker-compose logs gitlab

# 查看 GitLab 数据库日志
docker-compose exec gitlab gitlab-ctl tail

# 查看 GitLab Sidekiq 日志
docker-compose exec gitlab gitlab-ctl tail sidekiq
```

#### 查看 Jenkins 日志

```bash
# 查看 Jenkins 日志
docker-compose logs jenkins

# 查看 Jenkins 系统日志
docker-compose exec jenkins tail -f /var/log/jenkins/jenkins.log
```

### 2. 监控配置

#### 健康检查脚本

```bash
#!/bin/bash
# 健康检查脚本

echo "=== GitLab 服务状态 ==="
docker-compose exec gitlab gitlab-ctl status

echo "=== Jenkins 服务状态 ==="
curl -s http://jenkins.example.com:9080/api/json

echo "=== 磁盘使用情况 ==="
df -h

echo "=== 内存使用情况 ==="
free -h

echo "=== Docker 容器状态 ==="
docker-compose ps
```

#### Prometheus 监控（可选）

```yaml
# docker-compose 监控配置
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - '9090:9090'
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      '--storage.tsdb-path=/prometheus'
    networks:
      - gitlab-jenkins-net

  grafana:
    image: grafana/grafana:latest
    ports:
      - '3000:3000'
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - ./grafana/provisioning/datasources
      - ./grafana/provisioning/dashboards
    networks:
      - gitlab-jenkins-net
```

## 十、故障排除

### 1. 常见启动问题

#### 容器启动失败

```bash
# 检查容器状态
docker-compose ps

# 查看详细错误信息
docker-compose logs gitlab
docker-compose logs jenkins

# 检查端口占用
netstat -tunlp | grep -E ':(80|443|22|9080|50000)'

# 检查磁盘空间
df -h
```

#### 内存不足错误

```bash
# 检查内存使用
free -h

# 调整 GitLab 内存限制
# 在 docker-compose.yml 中添加
# environment:
#   GITLAB_OMNIBUS_CONFIG: |
#     unicorn['worker_memory_limit'] = 1024
```

#### 数据库连接问题

```bash
# 检查数据库连接
docker-compose exec gitlab gitlab-rake db:migrate:status

# 重置数据库连接
docker-compose exec gitlab gitlab-ctl reconfigure
```

### 2. 网络连接问题

#### 容器间通信失败

```bash
# 检查网络连通性
docker-compose exec gitlab ping jenkins
docker-compose exec jenkins ping gitlab

# 检查网络配置
docker network ls
docker network inspect gitlab-jenkins-net
```

#### 外部访问问题

```bash
# 检查防火墙状态
ufw status

# 检查端口监听
netstat -tunlp | grep -E ':(80|443|22|9080|50000)'
```

### 3. 权限问题

#### 文件权限错误

```bash
# 检查文件权限
ls -la /var/opt/gitlab/
ls -la /var/jenkins_home/

# 修复权限问题
docker-compose exec gitlab chown -R gitlab:gitlab /var/opt/gitlab/
docker-compose exec jenkins chown -R jenkins:jenkins /var/jenkins_home/
```

#### Docker Socket 权限

```bash
# 检查 Docker socket 权限
ls -la /var/run/docker.sock

# 修复权限问题
sudo chmod 666 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock

# 重启 Docker 服务
sudo systemctl restart docker
```

## 十一、安全配置

### 1. 网络安全

#### SSL/TLS 配置

```yaml
# GitLab HTTPS 配置
external_url 'https://gitlab.example.com'
gitlab_rails['gitlab_https'] = true
gitlab_rails['gitlab_port'] = 443
gitlab_rails['gitlab_ssl_certificate'] = '/etc/gitlab/ssl/gitlab.crt'
gitlab_rails['ssl_certificate_key'] = '/etc/gitlab/ssl/gitlab.key'
```

#### 防火墙配置

```bash
# 开放必要端口
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 22/tcp
ufw allow 9080/tcp
ufw allow 50000/tcp

# 查看防火墙规则
ufw status
```

### 2. 访问控制

#### SSH 密钥配置

```bash
# 生成 SSH 密钥
ssh-keygen -t rsa -b 4096 -C "gitlab@example.com" -f ~/.ssh/id_rsa

# 配置 GitLab SSH 密钥
docker-compose exec gitlab gitlab-rails runner 'ssh-keyscan -t rsa' < ~/.ssh/id_rsa.pub
```

#### LDAP 安全配置

```yaml
# LDAP 安全配置
gitlab_rails['ldap_allow_username_or_email_lower'] = false
gitlab_rails['ldap_block_anonymous_users'] = true
gitlab_rails['ldap_active_directory'] = 'ou=Users,ou=Groups'
gitlab_rails['ldap_group_base'] = 'ou=Groups,dc=example,dc=com'
gitlab_rails['ldap_user_filter'] = '(&(objectClass=user)(objectCategory=person)(!(memberOf=cn=disabled))
```

### 3. 数据加密

#### 数据库加密

```yaml
# PostgreSQL 加密配置
postgresql['data_dir'] = '/var/opt/gitlab/postgresql/data'
postgresql 'pgcrypto' = 'aes256'
```

#### 敏感信息保护

```yaml
# 敏感信息保护
gitlab_rails['gitlab_encrypted_db_credentials'] = true
gitlab_rails['gitlab_encrypted_secrets'] = true
```

## 十二、性能优化

### 1. 资源优化

#### 内存配置

```yaml
# GitLab 内存优化
environment:
  GITLAB_OMNIBUS_CONFIG: |
    # PostgreSQL 内存
    postgresql['shared_buffers'] = "256MB"
    postgresql['effective_cache_size'] = "1GB"
    
    # Redis 内存
    redis['maxmemory'] = '2gb'
    redis['maxmemory-policy'] = 'allkeys-lru'
    
    # Unicorn 内存
    unicorn['worker_memory_limit'] = 1024
    unicorn['worker_memory_limit_mb'] = 1024
```

#### 并发配置

```yaml
# Sidekiq 并发配置
sidekiq['concurrency'] = 10
sidekiq['min_concurrency'] = 5
sidekiq['max_concurrency'] = 20
```

### 2. 存储优化

#### 数据库存储

```yaml
# PostgreSQL 存储优化
gitlab_rails['db_pool_size'] = 20
gitlab_rails['db_database_tasks_concurrency'] = 4
```

#### 文件存储

```yaml
# 文件存储优化
gitlab_rails['gitlab_repository_downloads_path'] = '/var/opt/gitlab/shared'
gitlab_rails['gitlab_shared_path'] = '/var/opt/gitlab/shared'
```

## 总结

通过 Docker Compose 部署 GitLab 和 Jenkins，我们可以快速搭建一个完整的企业级 DevOps 平台。本文涵盖了从基础配置到高级优化的所有关键环节：

### 部署优势

- **快速部署**：一键启动完整的服务栈
- **环境隔离**：容器化部署避免环境冲突
- **数据持久化**：重要数据的安全存储
- **服务集成**：GitLab 与 Jenkins 无缝集成
- **可扩展性**：易于水平扩展

### 关键组件

- **GitLab CE 中文版**：支持中文界面和文档
- **Jenkins Blue Ocean**：现代化 CI/CD 平台
- **PostgreSQL**：可靠的数据存储
- **Redis**：高性能缓存服务
- **Nginx**：强大的反向代理

通过遵循本文的配置和最佳实践，可以构建一个稳定、高效、安全的 DevOps 工作流，为团队提供现代化的软件开发和部署能力。

---
*整理时间: 2026年2月10日*
*整理者: 来财 (OpenClaw AI助手)*
*参考来源: Docker Compose 最佳实践 + GitLab/Jenkins 官方文档*