---
title: "用frp实现内网穿透"
date: 2026-03-03T19:25:49+08:00
lastmod: 2026-03-03T14:40:15+08:00
---

## 简介
有时需要从公网访问内网的服务，这个时候就需要内网穿透了。有一些现成的产品可以用，但如果可以，干嘛不自己搭一个呢？
需要注意，自己搭必须要有一个公网 IP，如果没有，这篇文章对你没用。
其次，文章的服务端和客户端（内网中的服务）都是 ubuntu。

##  下载

先通过以下命令查看架构：

```bash
uname -a
```

下载页面： https://github.com/fatedier/frp/releases
根据架构来下载对应的安装包，比如 x86 就下载 amd64，arm 就下 arm，以此类推。然后更改 wget 命令后的下载链接。

```bash
wget https://github.com/fatedier/frp/releases/download/v0.67.0/frp_0.67.0_linux_amd64.tar.gz
tar -zxvf ./frp_0.67.0_linux_amd64.tar.gz -C /usr/local/
mv /usr/local/frp_0.67.0_linux_amd64 /usr/local/frp
cd /usr/local/frp
```

btw：如果 tar 命令提示找不到，可以试着手敲一遍。不知道为什么我复制命令然后粘贴会有这种问题。

## 服务端配置

接下来修改路径下的 `frps.toml`，根据自己的服务器配置修改：

```toml
# 服务端绑定端口（客户端连接用，默认7000，防火墙必须放行）
bindPort = 7000
# 认证密钥（客户端需保持一致，建议自定义强密码）
auth.token = "your_custom_token_123456"

# 可选：Web管理面板（方便查看连接状态，默认端口7500）
webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"       # 面板用户名
webServer.password = "panel_password" # 面板密码

# 可选：允许穿透的端口范围（限制客户端可用端口，提升安全性）
allowPorts = [
  { start = 10000, end = 65535 }
]
```

运行 `vim frps.toml` 修改文件，将以上内容粘贴进去。

运行以下命令手动测试：

```bash
./frps -c frps.toml
```

需要注意，如果是买的云服务器，得配置防火墙，放开配置文件中对应端口的限制。

## 客户端配置

接下来配置客户端。这里就不赘述下载的过程了，方法和之前一致。
修改 `frpc.toml` 文件，根据自己的情况进行修改：
 
```toml
# 服务端公网IP/域名
serverAddr = "1.2.3.4"
# 服务端绑定端口（与frps.toml的bindPort一致）
serverPort = 7000
# 认证密钥（与服务端完全一致）
auth.token = "your_custom_token_123456"

# 穿透规则1：SSH远程访问（外网访问服务端10022端口 → 内网22端口）
[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 10022

# 穿透规则2：Web服务（外网访问服务端10080端口 → 内网80端口）
[[proxies]]
name = "web"
type = "tcp"
localIP = "127.0.0.1"
localPort = 80
remotePort = 10080
```

运行以下命令手动测试（在 linux 下，windows 直接双击 exe）：

```bash
cd /usr/local/frp && ./frpc -c frpc.toml
```

## 小结
如果这个时候服务端和客户端都开启了，可以看到控制台中打印出的日志。
当然，也可以通过在服务端配置的 web 管理页面查看相应情况。

接着可以测试配置是否生效，比如验证 ssh：

```bash
# ssh 用户名@服务端IP -p 客户端配置的端口号，这里是10022
ssh root@192.168.1.5 -p 10022
```

至此，frp 就算安装完成了，接下来的内容属于锦上添花，若不需要，可以直接忽略。

## 服务端配置开机自启

运行命令：

```bash
sudo vim /etc/systemd/system/frps.service
```

粘贴以下内容，如果修改了安装目录，要做对应的修改：

```ini
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/usr/local/frp
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.toml
Restart=on-failure  # 异常崩溃自动重启
RestartSec=5s       # 重启间隔

[Install]
WantedBy=multi-user.target
```

接下来就可以愉快地用 `systemctl` 来管理服务了：

```bash
# 重新加载系统服务配置
sudo systemctl daemon-reload
# 设置开机自启
sudo systemctl enable frps
# 启动服务
sudo systemctl start frps
# 查看服务运行状态
sudo systemctl status frps
```

## 客户端配置开机自启

大致流程和服务端的一致。
如果是 windows，可以将 `frpc.exe` 的快捷方式放在“启动”目录下。

```bash
sudo vim /etc/systemd/system/frpc.service
```

粘贴配置文件：

```ini
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/usr/local/frp
ExecStart=/usr/local/frp/frpc -c /usr/local/frp/frpc.toml
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

管理命令：

```bash
# 重新加载系统服务配置
sudo systemctl daemon-reload
# 设置开机自启
sudo systemctl enable frpc
# 启动服务
sudo systemctl start frpc
# 查看服务运行状态
sudo systemctl status frpc
```

## 运维与删除

运维：

```bash
# 查看运行日志
sudo journalctl -u frps -f  # 服务端
sudo journalctl -u frpc -f  # 客户端

# 重启服务
sudo systemctl restart frps
# 停止服务
sudo systemctl stop frps
# 禁用开机自启
sudo systemctl disable frps
```

卸载：

```bash
# 停止并禁用服务
sudo systemctl stop frps frpc && sudo systemctl disable frps frpc
# 删除服务文件
sudo rm -f /etc/systemd/system/frp*.service
sudo systemctl daemon-reload
# 删除安装文件
sudo rm -rf /usr/local/frp
```

## 使用 SSH key 提升安全性
这一节实际跟 frp 关系不大，是 ssh 安全相关的配置，如不需要可跳过。

用公网 ssh 连接内网的电脑，总觉得光凭密码不够安全，这个时候就可以考虑使用 ssh key 来进行 ssh 身份验证了。

首先，在本地机器生成 SSH key（如果没有）。

```bash
# 这里的your_email@example.com，纯粹是方便整理的标识
ssh-keygen -t ed25519 -C "your_email@example.com"
```
  
然后将公钥复制到内网机器，可以使用命令：

```bash
# linux 下：
ssh-copy-id -p 你的SSH端口 user@你的frp域名

# windows 中 powershell 下：
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" | ssh -p 你的SSH端口 user@你的frp域名 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

或者手动复制

```bash
cat ~/.ssh/id_ed25519.pub
```

然后添加到内网机器的 `~/.ssh/authorized_keys` 文件中。如果目录下无此文件需新建。

---

接下来，需要配置 ssh，让它禁用密码登录、限制登录用户、失败次数等等，提升 ssh 的安全性。
编辑 `/etc/ssh/sshd_config` 文件：

```ini
# 禁用密码登录，只允许 key
PasswordAuthentication no
PubkeyAuthentication yes

# 禁用 root 登录
PermitRootLogin no

# 限制登录用户
AllowUsers your_username

# 更改默认端口（不要用22）
Port 22222

# 限制失败尝试
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
```

如果修改了端口，需要更改防火墙：

```bash
# 只允许必要端口
sudo ufw allow 22222/tcp  # SSH 端口
sudo ufw enable
```

重启 ssh 服务：

```bash
# CentOS/RHEL 7+ 
systemctl reload sshd
# Ubuntu/Debian 
systemctl reload ssh
```

---

ssh 的配置就算完成了，接下来可以安装 fail2ban 防止暴力破解：

```bash
# 安装
sudo apt install fail2ban
# 开启自启
sudo systemctl enable fail2ban
# 启动
sudo systemctl start fail2ban
```

## 总结
总的来说，frp 的安装还是很简单的。唯一有点恼人的是，我经常不知道自己配置成功没有，需要试上好几次。

除了本文说的，用二进制进行安装方式外，还可以使用 docker 进行安装，那样更简单，一行命令就能解决。
