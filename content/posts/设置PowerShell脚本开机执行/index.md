---
title: "设置PowerShell开机执行"
date: 2026-04-09T11:02:40+08:00
lastmod: 2026-04-09T11:02:40+08:00
tags: ["瞎折腾"]
---

通过任务计划程序实现，任务计划程序是最灵活且安全的方式，支持细粒度控制执行条件和权限。

## 1. 创建任务
1. 按 `Win + R`，输入 `taskschd.msc` 打开任务计划程序。
2. 右侧点击 **创建基本任务**，输入任务名称（如 `AutoRunScript`）和描述。
3. **触发器**选择 **当计算机启动时**，点击 **下一步**。（win11出现了不生效的情况，将条件改成了当用户登录时，成功生效）
4. **操作**选择 **启动程序**，点击 **下一步**。

## 2. 配置 PowerShell 参数
1. **程序或脚本**：输入 powershell 的路径。默认会是：`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

通过在 powershell 中输入 `$PSHOME` 查看安装路径。`_$PSHOME_` 是一个自动变量，存储了 PowerShell 的安装目录路径。

```
$PSHOME

示例输出：
C:\Program Files\PowerShell\7
```

那么这里就填

```
C:\Program Files\PowerShell\7\pwsh.exe
```

2. **添加参数**：输入以下内容（替换为实际脚本路径）：
```plaintext
-ExecutionPolicy Bypass -File "C:\路径\to\脚本.ps1"
```
- `-ExecutionPolicy Bypass` 临时绕过执行策略限制。
- `-File` 指定脚本文件路径，路径需用英文引号包裹。

## 3. 高级设置（可选）
1. 点击 **完成** 后，右键任务选择 **属性**。
2. **常规** 选项卡：
    - 勾选 **不管用户是否登录都要运行**。
    - 勾选 **使用最高权限运行**（若脚本需要管理员权限）。
3. **条件** 选项卡：
    - 取消勾选 **只有在计算机使用交流电源时才启动此任务**（适用于笔记本）。

## 4. 验证
1. 右键任务选择 **运行**，检查脚本是否正常执行。
2. 查看事件查看器（`eventvwr.msc`）的 **Windows 日志 → 系统**，搜索任务名称以确认执行状态。
