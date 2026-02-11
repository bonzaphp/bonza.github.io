---
layout: post
title:  "macOS 设置合盖不睡眠，远程 ssh 能够正常使用，但是某些系统中的软件会断联"
date:   2026-02-11 12:11:00 +0800
categories: macos
tags: macos ssh 电源管理 caffeinate pmset 合盖不睡眠
author: 来财
---


* content
{:toc




这是一个经典的 macOS 电源管理问题。合盖不睡眠设置确实能让 SSH 保持连接，但某些系统级网络组件和后台服务仍会被暂停或节流。

## 根本原因

macOS 的 **"Power Nap" / 现代待机模式** 即使在"不睡眠"状态下，也会：

- 暂停用户级网络进程
- 节流后台应用的网络访问
- 关闭显示器相关的 GPU 进程
- 某些系统服务进入低功耗模式

## 解决方案（按推荐程度排序）

### 方案 1：使用 `caffeinate` 命令（推荐，最干净）

```bash
# 防止系统睡眠，保持网络活跃
caffeinate -s -i -d -u -t 86400 &
# 或永久保持（直到手动取消）
caffeinate -s -i -d -u
```

参数说明：

- `-s`：防止系统睡眠（idle sleep）
- `-i`：防止系统空闲睡眠
- `-d`：防止显示器睡眠
- `-u`：模拟用户活动
- `-t 秒数`：指定持续时间

### 方案 2：修改 pmset 设置（持久化）

```bash
# 查看当前设置
pmset -g

# 禁用睡眠（插电时）
sudo pmset -c sleep 0
sudo pmset -c disablesleep 1

# 关闭 Power Nap
sudo pmset -c powernap 0

# 关闭待机模式（关键！）
sudo pmset -c standby 0
sudo pmset -c autopoweroff 0

# 设置休眠模式为 0（不写入内存镜像到磁盘，保持 RAM 供电）
sudo pmset -c hibernatemode 0
```

### 方案 3：TCP Keepalive + SSH 配置优化

编辑 `~/.ssh/config`：

```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes
    
# 针对特定主机
Host your-mac-host
    HostName your-mac-ip
    ServerAliveInterval 30
```

### 方案 4：禁用 App Nap（针对特定软件）

```bash
# 全局禁用 App Nap
defaults write NSGlobalDomain NSAppSleepDisabled -bool YES

# 或针对特定应用（以 Terminal 为例）
defaults write com.apple.Terminal NSAppSleepDisabled -bool YES
```

## 排查断联的具体软件

你可以用以下命令查看哪些进程被暂停：

```bash
# 查看电源断言状态
pmset -g assertions

# 查看被 App Nap 影响的应用
ps -eo pid,comm,state | grep -i nap
```

## 我的建议配置

**日常使用**（插电合盖远程办公）：

```bash
# 1. 插电时完全禁用睡眠
sudo pmset -c sleep 0 standby 0 autopoweroff 0 powernap 0

# 2. 使用 caffeinate 作为二次保险
nohup caffeinate -s -i -d -u > /dev/null 2>&1 &

# 3. SSH 配置优化 Keepalive
```

**注意事项**：

- 上述设置仅影响 **AC Power（插电）** 状态，电池模式仍正常睡眠
- 合盖时注意散热，MacBook 合盖时散热会变差
- 长期运行建议外接显示器或使用散热支架

------

**你具体是哪些软件断联？** 如果是 Docker、VPN 客户端或特定开发工具，可能有更针对性的解决方案。

`caffeinate -s -i -d -u`

这句是不是显示器一直是开启状态



`caffeinate -d` 确实会阻止显示器睡眠，但**合盖时显示器实际上是物理关闭的**，所以这个参数在合盖场景下效果有限。

让我澄清一下合盖时的实际行为：

## 合盖时的真实情况

| 参数           | 作用           | 合盖时是否有效               |
| :------------- | :------------- | :--------------------------- |
| `-s` (system)  | 防止系统睡眠   | ✅ **有效，核心需求**         |
| `-i` (idle)    | 防止空闲睡眠   | ✅ 有效                       |
| `-d` (display) | 防止显示器睡眠 | ❌ **无效**（屏幕已物理关闭） |
| `-u` (user)    | 模拟用户活动   | ⚠️ 部分有效（重置空闲计时器） |

## 优化后的命令

**合盖远程专用（推荐）：**

```bash
# 精简版：只需阻止系统级睡眠
caffeinate -s -i

# 或加上模拟用户活动，防止某些服务因"无用户活动"被暂停
caffeinate -s -i -u
```

`-d` 在合盖时确实多余，但留着也没副作用——它只是向系统声明"有个进程需要显示器保持开启"，不会真的让合盖的屏幕亮起来。

## 更精准的问题定位

如果你的软件断联，可能不是睡眠问题，而是：

```bash
# 检查合盖后网络接口是否被降级
ifconfig | grep -A 2 "en0\|en1"

# 查看 WiFi 电源管理状态（合盖后可能变为 ON）
sudo pmset -g | grep womp
```

**`womp` (Wake On Magic Packet)** 如果开启，合盖后网卡可能进入低功耗模式。

## 终极合盖不掉线配置

```bash
# 1. 插电时完全禁止睡眠 + 关闭网卡低功耗
sudo pmset -c sleep 0 standby 0 autopoweroff 0 powernap 0 womp 0

# 2. 保持 caffeinate 运行（无需 -d）
caffeinate -s -i -u &

# 3. 如果软件仍断联，检查是否是 App Nap 导致
sudo pmset -g assertions | grep -i "applepushserviced\|apsd"
```

------

**你的软件断联有规律吗？** 比如：

- 固定时间后断开（15分钟/30分钟）→ 可能是 TCP 超时或 App Nap
- 随机断开 → 可能是网络接口电源管理
- 特定软件（如 Docker、数据库、VPN）→ 需要针对性配置

```
$ pmset -g | grep -E "standby|autopoweroff|hibernatemode|powernap"
 autopoweroff         0
standbydelayhigh     86400
autopoweroffdelay    259200
standbydelaylow      10800
standby              0
hibernatemode        0
powernap             0
highstandbythreshold 50
```

我看到你的配置了，问题很明显：

## 关键问题点

| 参数            | 当前值 | 问题                             |
| :-------------- | :----- | :------------------------------- |
| `standby`       | **1**  | ✅ **合盖后进入待机，SSH 会断**   |
| `autopoweroff`  | **1**  | ✅ **长时间后自动断电**           |
| `hibernatemode` | **3**  | ✅ **安全睡眠模式，内存写入磁盘** |
| `powernap`      | **1**  | 后台活动可能干扰网络             |
| `sleep`         | 0      | 仅延迟睡眠，但 standby 会覆盖    |

**核心问题：`standby 1` + `hibernatemode 3`** —— 这是 macOS 的"安全睡眠"组合，合盖后即使插电，一段时间后也会把内存写入磁盘并进入低功耗状态，网络必然中断。

## 修复方案

### 第一步：关闭待机模式（关键）

```bash
# 插电时完全禁止待机和自动断电
sudo pmset -c standby 0
sudo pmset -c autopoweroff 0
sudo pmset -c hibernatemode 0
sudo pmset -c powernap 0

# 可选：关闭网卡唤醒（防止网卡进入低功耗）
sudo pmset -c womp 0
```

### 第二步：验证修改

```bash
pmset -g | grep -E "standby|autopoweroff|hibernatemode|powernap"
```

应该显示：

```
standby              0
autopoweroff         0
hibernatemode        0
powernap             0
```

### 第三步：启动 caffeinate（二次保险）

```bash
# 后台运行，防止系统空闲
nohup caffeinate -s -i -u > /dev/null 2>&1 &
```

## 关于 `hibernatemode` 的说明

| 模式 | 行为                   | 适用场景              |
| :--- | :--------------------- | :-------------------- |
| 0    | 普通睡眠，RAM 保持供电 | ✅ **服务器/远程访问** |
| 3    | 安全睡眠，RAM+磁盘镜像 | 默认，断电不丢数据    |
| 25   | 强制休眠，类似 Windows | 省电优先              |

设置为 `0` 后，合盖时仅 RAM 保持供电，恢复极快，且网络不中断。**注意：断电会丢失未保存数据**（但你是插电使用，风险极低）。

## 修改后的理想配置

```
standby              0
autopoweroff         0
hibernatemode        0
powernap             0
sleep                0
womp                 0
ttyskeepawake        1  ✅ （保持 TTY/SSH 唤醒）
tcpkeepalive         1  ✅ （TCP 保活）
```

------

**修改后立即生效，无需重启**。合盖测试 SSH 是否还断联？

如果仍有问题，可能是特定软件的 **App Nap** 或 **Background App Refresh** 导致，可以进一步排查。