---
layout: post
title:  "jenkins配置Publish over SSH的小问题"
date:   2026-02-11 02:15:00 +0800
categories: jenkins
tags: jenkins ssh key publish over ssh
author: 来财
---


* content
{:toc}

# 目录




本文介绍了 Jenkins 配置 Publish over SSH 插件时遇到的 SSH 密钥格式兼容性问题，以及通过降级密钥版本格式来解决问题的具体方法。

## jenkins配置Publish over SSH的小问题

***

### 问题描述：

最近配置jenkins出现了个问题，Publish over SSH 的key明明是正确的，但总是报以下错：

    jenkins.plugins.publish_over.BapPublisherException: Failed to add SSH key. Message [invalid privatekey: [B@5216eae]

#### 原因分析：

经过折腾终于发现问题：
id\_rsa版本太新了！！！我的Jenkins 是2.326，id\_rsa开头是：-----BEGIN OPENSSH PRIVATE KEY-----
jenkins2.326检验开头还不支持该格式

解决方案：
给id\_rsa降版本：
使用该指令：

    ssh-keygen -m PEM -t rsa -b 4096

生成开头为：

    -----BEGIN RSA PRIVATE KEY-----

测试：SUCCESS
————————————————