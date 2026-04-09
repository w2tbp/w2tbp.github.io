---
title: "Syncthing"
date: 2022-11-02T22:51:55+08:00
lastmod: 2026-04-07T10:50:11+08:00
tags: ["瞎折腾"]
---

这个东西用来同步 obsidian 的笔记，用了好几年，还是很稳定的。

之前使用的是微力同步，然后发现是 Syncthing 的套壳，就改成用 Syncthing 了。

需要注意的是，本文只介绍了怎么安装，如想知道如何使用可以参考：[Syncthing - P2P文件同步工具](https://zhuanlan.zhihu.com/p/69267020)

## window 下安装
### 新方法
由于 win11 更新，之前的方法不管用了，于是使用了新的方法，使用 powershell 脚本。

```powershell
# 隐藏窗口运行程序的PowerShell脚本

# 替换为你的exe完整路径
$exePath = "D:\software\syncthing-windows-amd64-v1.29.6\syncthing.exe"

# 替换为实际参数
$arguments = "serve --no-browser --no-restart --logflags=0"

# 检查文件是否存在
if (-not (Test-Path -Path $exePath -PathType Leaf)) {
    Write-Error "未找到文件: $exePath"
    exit 1
}

# 启动进程并传递参数，隐藏窗口
Start-Process -FilePath $exePath -ArgumentList $arguments -WindowStyle Hidden

# 可选：如果需要等待程序执行完成再退出，添加 -Wait 参数
# Start-Process -FilePath $exePath -ArgumentList $arguments -WindowStyle Hidden -Wait
```

设置脚本开机自启：[设置PowerShell脚本开机执行](https://w2tbp.github.io/posts/设置powershell脚本开机执行)

来源：
- [PowerShell 实际应用 | 菜鸟教程](https://www.runoob.com/powershell/powershell-practice.html)
- [发现 PowerShell - PowerShell | Microsoft Learn](https://learn.microsoft.com/zh-cn/powershell/scripting/discover-powershell?view=powershell-7.5)
- 发现别人也有同样的问题，不推荐使用 vbs 了：[Win11 25h2 7105 mshta有问题了？ - 远景论坛 - 前沿科技与智慧生态的极客社区 -](https://bbs.pcbeta.com/viewthread-2055319-1-1.html)

### 旧方法 v2
可能是 syncthing 的新版本原因，也有可能是因为 win11 原因，总之以前的方法不管用了，在网上找到了一个大佬的方法，尝试过后发现确实可以。

在syncthing的安装目录下新建一个bat文件，录入以下内容后保存。

```bat
@echo off
 
if "%1"=="h" goto begin
 
start mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
 
:begin
 
cd /d D:\Program Files\Syncthing && syncthing.exe serve --no-browser --no-restart --logflags=0
```

注意bat脚本中最后一行中的文件路径 `D:\Program Files\Syncthing` 要根据自己的实际情况调整。

然后把 bat 脚本的快捷方式丢到开机自启当中。

-  [Syncthing基础使用：在Windows下设置开机自启 & 后台运行 // 喵ฅ^•ﻌ•^ฅ](https://ruohai.wang/202411/syncthing-auto-start-and-run-backend-on-windows/)

### 旧方法 v1
有客户端版本的，由社区维护，我下下来试了下，没有跟到最新版本，看其他文章好像有 bug ，而且大佬也给出了其他的方案。

也就是开机自启一个 bat 文件，启动 Syncthing 提供的命令行工具。

先去官网下个包 [https://syncthing.net/downloads/](https://syncthing.net/downloads/)
![[syncthing-1.png]]

然后 win + r 输入 shell:startup ，在其中新建一个 bat 文件（可以先新建个 txt 文件，输入下面内容后，再将 txt 后缀改为 bat）

需要修改三个地方

1. `D:\software\syncthing-windows-amd64-v1.22.0\syncthing.exe`

程序的目录。

2. `-config="C:\Users\admin\AppData\Local\Syncthing"`

Syncthing 的配置目录。得先运行一下安装包的 syncthing.exe 才会出现，下面那个也一样

3. `-data="C:\Users\admin\AppData\Local\Syncthing\index-v0.14.0.db"`

```
@ECHO OFF
%1 start mshta vbscript:createobject("wscript.shell").run("""%~0"" ::",0)(window.close)&&exit
start /b D:\software\syncthing-windows-amd64-v1.22.0\syncthing.exe -config="C:\Users\admin\AppData\Local\Syncthing" -data="C:\Users\admin\AppData\Local\Syncthing\index-v0.14.0.db" -no-browser
```

程序运行后，访问 [http://127.0.0.1:8384/](http://127.0.0.1:8384/) 可以进入管理页面

参考链接

[Syncthing文件同步方案完全攻略（亲测有效）_rockage的博客-CSDN博客_syncthing](https://blog.csdn.net/rockage/article/details/121079720)

## ubuntu 下安装

同步服务，就需要一个24小时在线的服务器。我用的是腾讯云的，系统是 Ubuntu Server 20.04 LTS

```bash
# Add the release PGP keys:
curl -s https://syncthing.net/release-key.txt | gpg --dearmor | sudo tee /usr/share/keyrings/syncthing-archive-keyring.gpg >/dev/null

# Add the "stable" channel to your APT sources:
echo "deb [signed-by=/usr/share/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list

# Update and install syncthing:
sudo apt-get update
sudo apt-get install syncthing

# check version
syncthing --version
```

上面安装好了，但是会有个问题，这玩意的 webui 不能用服务器的公网 ip 访问，只能用 [http://127.0.0.1:8384/](http://127.0.0.1:8384/) 访问。应该可以用 nginx 的反向代理解决，我太菜了配置半天没成，然后发现了另外一种方法。

以下来源参考链接1

自 Ubuntu 18.04+ 开始，就可以通过创建 systemd 配置文件来管理 syncthing 服务。官方也提供了配置文件：[etc/linux-systemd](https://github.com/syncthing/syncthing/tree/master/etc/linux-systemd)

首先先创建个文件

`sudo vim /etc/systemd/system/syncthing@.service`

然后输入以下内容

```
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization for %I
Documentation=man:syncthing(1)
After=network.target

[Service]
User=%i
ExecStart=/usr/bin/syncthing -no-browser -gui-address="0.0.0.0:8384" -no-restart -logflags=0
Restart=on-failure
SuccessExitStatus=3 4
RestartForceExitStatus=3 4

[Install]
WantedBy=multi-user.target
```

这样，你就能用腾讯云给的 ip 来访问了，当然防火墙得开 8384 。

注意：并且需要开 22000 端口，不然添加不了远程设备（我被坑半天）

然后可以配置一下 systemd

```
# 更新 systemd 服务
sudo systemctl daemon-reload

# 启动 syncthing 服务
sudo systemctl start syncthing@$USER
# 开启自启
sudo systemctl enable syncthing@$USER

# 查看服务状态
systemctl status syncthing@$USER

# 重启
systemctl restart syncthing@$USER
```

参考链接

1. [https://computingforgeeks.com/how-to-install-and-use-syncthing-on-ubuntu/](https://computingforgeeks.com/how-to-install-and-use-syncthing-on-ubuntu/)
2. [https://apt.syncthing.net/](https://apt.syncthing.net/)


## 安卓

安卓可以自行下载 APP 使用。

**如果安卓手机端同步目录提示错误 (folder marker missing)** 

这是由于该同步目录下面缺少一个 .stfolder 目录，解决办法是在该目录下新建文件夹:.stfolder (注意前面的 ".")，因为该文件夹为隐藏文件夹，有的国内定制安卓系统或者系统清理软件会自动清除该文件夹，所以如果新建 .stfolder 文件夹后还出现这样的情况，可以在 .stfolder 里随便新建一个空文件，比如我就在该文件夹下新建一个名为 .stfolder 的空文件。

- [https://zhuanlan.zhihu.com/p/121544814](https://zhuanlan.zhihu.com/p/121544814)
