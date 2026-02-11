---
layout: post
title:  "docker-compose 一键部署 rsshub"
date:   2026-02-11 02:42:00 +0800
categories: docker
tags: docker-compose rsshub redis browserless 部署
author: 来财
---


* content
{:toc}

# 目录

# Docker Compose 一键部署 RSSHub




本文提供了使用 Docker Compose 一键部署 RSSHub 的完整配置文件，包含了 RSSHub 主服务、Redis 缓存、Browserless 浏览器服务和真实浏览器服务的完整部署方案。

### docker-compose 一键部署 rsshub

***

    services:
        rsshub:
            # two ways to enable puppeteer:
            # * comment out marked lines, then use this image instead: diygod/rsshub:chromium-bundled
            # * (consumes more disk space and memory) leave everything unchanged
            image: diygod/rsshub # or ghcr.io/diygod/rsshub
            restart: always
            ports:
                - '1200:1200'
            environment:
                NODE_ENV: production
                CACHE_TYPE: redis
                REDIS_URL: 'redis://redis:6379/'
                PUPPETEER_WS_ENDPOINT: 'ws://browserless:3000' # marked
                PUPPETEER_REAL_BROWSER_SERVICE: 'http://real-browser:3000' # marked
            healthcheck:
                test: ['CMD', 'curl', '-f', 'http://localhost:1200/healthz']
                interval: 30s
                timeout: 10s
                retries: 3
            depends_on:
                - redis
                - browserless # marked

        real-browser:
            image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/ghcr.io/hyoban/puppeteer-real-browser-hono:latest
            restart: always
            ports:
                - '3001:3000'
            healthcheck:
                test: ['CMD', 'curl', '-f', 'http://localhost:3000']
                interval: 30s
                timeout: 10s
                retries: 3

        browserless: # marked
            image: browserless/chrome # marked
            restart: always # marked
            ulimits: # marked
                core: # marked
                    hard: 0 # marked
                    soft: 0 # marked
            healthcheck: # marked
                test: ['CMD', 'curl', '-f', 'http://localhost:3000/pressure'] # marked
                interval: 30s # marked
                timeout: 10s # marked
                retries: 3 # marked

        redis:
            image: redis:alpine
            restart: always
            volumes:
                - redis-data:/data
            healthcheck:
                test: ['CMD', 'redis-cli', 'ping']
                interval: 30s
                timeout: 10s
                retries: 5
                start_period: 5s

    volumes:
        redis-data: