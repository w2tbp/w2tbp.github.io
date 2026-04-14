---
title: "wsl使用"
date: 2026-04-14T09:41:48+08:00
lastmod: 2026-04-14T09:41:48+08:00
---

## win11 下安装

### 安装 wsl
直接运行：

```bash
wsl --install
```

会自动开启 **适用于 Linux 的 Windows 子系统** 和 **虚拟机平台** 两个功能。

ps：之前折腾坏了，**虚拟机平台** 死都起不来，最后重装系统才好。

### 安装发行版

```bash
# 查看线上有哪些可用的发行版
wsl --list --online

# 安装 Ubuntu
wsl --install -d Ubuntu

# 安装到指定位置
wsl --install -d Ubuntu --location D:\wsl\Ubuntu
```

### 常用命令

```bash
# 查看有哪些实例
wsl -l -v

# 启动实例
wsl -d 实例名称
wsl -d Ubuntu-22.04

# 暂停实例
wsl --terminate <实例名称> 
wsl --terminate Ubuntu-22.04 
# 或简写 
wsl -t Ubuntu-22.04

# 暂停所有运行中的 WSL 实例
wsl --shutdown
```

卸载发行版：

```bash
wsl --unregister ubuntu
```