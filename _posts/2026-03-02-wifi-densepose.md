---
layout: post
title: "WiFi DensePose：用WiFi信号透视墙壁的革命性技术"
date: 2026-03-02 15:19:00 +0800
categories: 开源项目
tags: WiFi AI 计算机视觉 隐私保护 开源项目
author: 来财
---

WiFi DensePose 让我们能够通过WiFi信号"看穿"墙壁。无需摄像头，无需穿戴设备，仅靠无线电波就能实现实时人体姿态估计、生命体征监测和存在检测。这是一个突破性的开源项目，它将普通的WiFi信号转化为实时的人体感知能力。




* content
{:toc}

## 项目概述

WiFi DensePose 是一个突破性的开源项目，它将普通的WiFi信号转化为实时的人体感知能力。通过分析人体运动引起的信道状态信息（CSI）扰动，系统能够重建身体位置、呼吸频率和心率，所有这些都基于物理信号处理和机器学习技术。

## 核心能力

### 🎯 感知能力

- **姿态估计**：CSI子载波幅度/相位 → DensePose UV映射，处理速度达54K fps（Rust实现）
- **呼吸检测**：带通滤波0.1-0.5 Hz → FFT峰值检测，支持6-30 BPM呼吸频率
- **心率监测**：带通滤波0.8-2.0 Hz → FFT峰值检测，支持40-120 BPM心率范围
- **存在感知**：RSSI方差 + 运动频带功率，延迟小于1ms
- **穿墙探测**：菲涅尔区几何 + 多径建模，深度可达5米

### 🧠 智能特性

- **隐私优先**：仅使用WiFi信号追踪人体姿态，不存储任何视频或图像
- **多人追踪**：同时追踪多个人，每个人都有独立的姿态和生命体征数据
- **自学习系统**：从原始WiFi数据中自主学习，无需标记数据集或摄像头引导
- **跨环境泛化**：训练一次，在任何房间都能部署，无需重新校准

## 技术架构

### 信号处理流程

WiFi路由器 → 无线电波穿透房间 → 碰到人体 → 散射
    ↓
ESP32/WiFi网卡捕获56+个子载波幅度和相位（CSI），频率20Hz
    ↓
信号处理清理噪声、去除干扰、提取运动特征
    ↓
AI骨干网络（RuVector）应用注意力机制、图算法和智能压缩
    ↓
神经网络将处理后的信号映射到17个身体关键点 + 生命体征
    ↓
输出：实时姿态、呼吸频率、心率、存在状态、房间指纹

### 硬件支持

| 硬件选项 | 设备 | 成本 | 完整CSI | 功能 |
|---------|------|------|---------|------|
| ESP32 Mesh（推荐） | 3-6个ESP32-S3 + WiFi路由器 | ~$54 | 是 | 姿态、呼吸、心率、运动、存在 |
| 研究级网卡 | Intel 5300 / Atheros AR9580 | ~$50-100 | 是 | 完整CSI + 3x3 MIMO |
| 任意WiFi设备 | Windows、macOS或Linux笔记本 | $0 | 否 | 仅RSSI：粗略存在和运动检测 |

## 应用场景

### 🏥 医疗健康
- **老年人护理**：跌倒检测、夜间活动监测、睡眠呼吸监测
- **医院患者监护**：非重症床位连续呼吸+心率监测
- **急诊分诊**：自动占用计数+等待时间估计

### 🏬 商业零售
- **零售占用率**：实时人流量、区域停留时间、排队长度
- **办公空间利用**：实际占用情况、会议室未出现情况、HVAC优化
- **酒店餐饮**：房间占用、迷你吧/洗手间使用模式、空房间节能

### 🤖 机器人与工业
- **协作机器人安全区**：检测协作机器人附近的人员存在
- **仓库AMR导航**：自主移动机器人感知盲角人员
- **制造监控**：工作站人员存在、人体工程学姿势警报

### 🚨 灾难救援
- **搜救行动**：通过碎石瓦砾检测幸存者、START伤情分类、3D定位
- **消防**：在烟雾和墙壁中定位人员、远程确认生命体征

## 性能优势

### 🦀 Rust实现（v2）—— 810倍速度提升

| 操作 | Python (v1) | Rust (v2) | 加速比 |
|------|-------------|-----------|--------|
| CSI预处理 (4x64) | ~5ms | 5.19 µs | ~1000x |
| 相位清理 (4x64) | ~3ms | 3.84 µs | ~780x |
| 特征提取 (4x64) | ~8ms | 9.03 µs | ~890x |
| 运动检测 | ~1ms | 186 ns | ~5400x |
| 完整流程 | ~15ms | 18.47 µs | ~810x |

### 📊 资源对比

| 资源 | Python (v1) | Rust (v2) |
|------|-------------|-----------|
| 内存 | ~500 MB | ~100 MB |
| Docker镜像 | 569 MB | 132 MB |
| 测试数量 | 41 | 542+ |
| WASM支持 | 否 | 是 |

## 快速开始

### Docker部署（推荐）

```bash
# 拉取镜像
docker pull ruvnet/wifi-densepose:latest

# 启动服务
docker run -p 3000:3000 -p 3001:3001 -p 5005:5005/udp ruvnet/wifi-densepose:latest

# 打开 http://localhost:3000
```

### 从源码构建

```bash
git clone https://github.com/ruvnet/wifi-densepose.git
cd wifi-densepose

# Rust（主要版本）
cd rust-port/wifi-densepose-rs
cargo build --release
cargo test --workspace

# 启动服务器
./target/release/sensing-server --source simulate --http-port 3000 --ws-port 3001
```

### API使用

```python
from wifi_densepose import WiFiDensePose

system = WiFiDensePose()
system.start()
poses = system.get_latest_poses()
print(f"检测到 {len(poses)} 个人")
system.stop()
```

## REST API

```bash
# 健康检查
curl http://localhost:3000/health

# 最新感知帧
curl http://localhost:3000/api/v1/sensing/latest

# 生命体征
curl http://localhost:3000/api/v1/vital-signs

# 姿态估计
curl http://localhost:3000/api/v1/pose/current
```

## 项目特色

### 🔬 独立能力审计（ADR-028）
- 1031项测试全部通过
- SHA-256证明，自验证见证包
- 33行证明矩阵：31项能力已验证

### 🧠 自学习WiFi AI（ADR-024）
- 对比自监督学习，房间指纹识别
- 异常检测，个人重识别
- 55KB模型，适合ESP32部署

### 🌍 跨环境泛化（ADR-027）
- 梯度反转层，几何编码器
- 硬件标准化，虚拟域增强
- 零样本部署，快速适应

## 开源与许可

WiFi DensePose 采用MIT许可证开源，由ruvnet开发。项目自2025年3月开始积极开发，经历了3个主要版本发布，提供了Rust移植版、最先进的信号处理、灾难响应模块和端到端训练管道。

## 总结

WiFi DensePose 代表了无线感知技术的重大突破。它不仅提供了前所未有的隐私保护人体感知能力，还通过开源的方式让这项技术能够被广泛应用。从医疗健康到商业零售，从机器人技术到灾难救援，这项技术都有着巨大的应用潜力。

随着硬件成本的不断降低和算法的持续优化，我们有理由相信，WiFi信号将成为下一代智能感知系统的重要组成部分，为我们的生活带来更多便利和安全保障。

---

*项目地址：https://github.com/ruvnet/wifi-densepose*