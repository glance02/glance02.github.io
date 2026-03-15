---
layout:     post
title:      "frp 内网穿透配置指南"
subtitle:   "使用 frp 实现公网访问本地服务"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 网络服务
---

## 一、环境架构

```
[任意设备] 
      ↓ SSH/HTTP 访问 公网地址:端口
[Ubuntu 公网服务器: frps] ←----加密隧道----→ [Windows 电脑: frpc]
                                              ↓
                                         [本地服务: Python HTTP/SSH]
```

## 二、服务端配置（Ubuntu 公网服务器）

### 1. 配置文件 `frps.toml`

```toml
# 基础通信端口（客户端连这个）
bindPort = 7000
# bindAddr = "0.0.0.0"  # 一般来说这个默认就是所有地址可以访问

# 安全认证（必须！）
auth.method = "token"
auth.token = "your_secure_token_123456"

# 管理面板（可选，用于看连接状态）
webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "0000l"

# 强制加密传输
# transport.tls.force = true
```

### 2. 启动方式

**前台调试（看日志）：**
```bash
cd /path/to/frp
./frps -c frps.toml
# 显示 "frps started successfully" 后保持窗口运行
```

**后台常驻（生产环境）：**
```bash
# 使用 systemd
sudo tee /etc/systemd/system/frps.service > /dev/null <<EOF
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5s
ExecStart=/path/to/frp/frps -c /path/to/frp/frps.toml

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable frps
sudo systemctl start frps
```

### 3. 防火墙/安全组必开端口

| 端口             | 用途                     | 来源                  |
| -------------- | ---------------------- | ------------------- |
| **7000**       | frp 通信端口               | 客户端 IP（或 0.0.0.0/0） |
| **7500**       | 管理面板（可选）               | 你的办公 IP             |
| **6000-13000** | 映射的服务端口（如 8080, 12059） | 0.0.0.0/0           |

```bash
# Ubuntu 本地防火墙，我一般直接不开服务器内部的防火墙
sudo ufw allow 7000/tcp
sudo ufw allow 7500/tcp
sudo ufw allow 6000:13000/tcp
```

---

## 三、客户端配置（Windows 电脑）

### 1. 配置文件 `frpc.toml`
```toml
# 服务端信息（你的公网服务器）
serverAddr = "你的公网ip"
serverPort = 7000
auth.token = "your_secure_token_123456"

# 客户端标识（多设备区分）
user = "home-pc"

# ========== 服务 1：HTTP 测试（Python 文件服务器）==========
[[proxies]]
name = "web-test"
type = "tcp"
localIP = "127.0.0.1"      # 本地服务 IP
localPort = 8888           # Python http.server 的端口
remotePort = 8080          # 外网访问：http://121.43.249.54:8080

# ========== 服务 2：WSL2 SSH（穿透 SSH）==========
[[proxies]]
name = "wsl-ssh"
type = "tcp"
# WSL2 注意：如果 frpc 跑在 Windows 上，需先端口转发 WSL2 的 22 到 Windows 的 2222
# 命令：netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=<WSL2_IP>
localIP = "127.0.0.1"
localPort = 2222           # 本地监听端口（WSL2 转发的端口）
remotePort = 12059         # 外网访问：ssh -p 12059 user@121.43.249.54
transport.useEncryption = true  # 额外加密层

# ========== 服务 3：Windows RDP（远程桌面）==========
[[proxies]]
name = "rdp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3389           # Windows 远程桌面默认端口
remotePort = 13389         # 外网访问：mstsc -> 121.43.249.54:13389
```

### 2. 启动方式

**PowerShell / CMD：**
```powershell
cd E:\Applications\frp_0.67.0_windows_amd64
.\frpc.exe -c frpc.toml
```

**后台运行（Windows）：**
```powershell
# 使用 NSSM 包装成服务
nssm install frpc E:\Applications\frp_0.67.0_windows_amd64\frpc.exe
nssm set frpc Application E:\Applications\frp_0.67.0_windows_amd64\frpc.exe
nssm set frpc AppParameters "-c frpc.toml"
nssm set frpc AppDirectory E:\Applications\frp_0.67.0_windows_amd64
nssm start frpc
```

---

## 五、最常用配置参数速查表

### 服务端（frps.toml）

| 参数                    | 类型     | 说明                  | 示例                         |
| --------------------- | ------ | ------------------- | -------------------------- |
| `bindPort`            | int    | 客户端连接端口             | `7000`                     |
| `bindAddr`            | string | 监听地址                | `"0.0.0.0"`                |
| `auth.method`         | string | 认证方式：`token`/`oidc` | `"token"`                  |
| `auth.token`          | string | 连接密码                | `"password123"`            |
| `transport.tls.force` | bool   | 强制 TLS 加密           | `true`                     |
| `webServer.port`      | int    | 管理面板端口              | `7500`                     |
| `allowPorts`          | array  | 允许映射的端口范围           | `[{start=6000, end=7000}]` |
| `maxPoolCount`        | int    | 最大连接池数量             | `50`                       |
| `tcpMux`              | bool   | TCP 多路复用            | `true`                     |

### 客户端（frpc.toml）

| 参数                                 | 类型     | 说明                           | 示例                    |
| ---------------------------------- | ------ | ---------------------------- | --------------------- |
| `serverAddr`                       | string | 服务端 IP/域名                    | `"121.43.249.54"`     |
| `serverPort`                       | int    | 服务端 bindPort                 | `7000`                |
| `auth.token`                       | string | 连接密码                         | `"password123"`       |
| `user`                             | string | 用户标识（前缀）                     | `"home-pc"`           |
| `[[proxies]]`                      | table  | 代理配置块（可多个）                   | -                     |
| `proxies.name`                     | string | 服务唯一名称                       | `"ssh"`               |
| `proxies.type`                     | string | 类型：`tcp`/`udp`/`http`/`stcp` | `"tcp"`               |
| `proxies.localIP`                  | string | 本地服务地址                       | `"127.0.0.1"`         |
| `proxies.localPort`                | int    | 本地服务端口                       | `22`                  |
| `proxies.remotePort`               | int    | 远程暴露端口                       | `6000`                |
| `proxies.customDomains`            | array  | HTTP/HTTPS 专用域名              | `["nas.example.com"]` |
| `proxies.transport.useEncryption`  | bool   | 端到端加密                        | `true`                |
| `proxies.transport.useCompression` | bool   | 流量压缩                         | `true`                |
| `proxies.secretKey`                | string | STCP 类型连接密钥                  | `"secret123"`         |

### 特殊类型：STCP（点对点加密）

**被访问端（暴露服务）：**
```toml
[[proxies]]
name = "secret-web"
type = "stcp"
localIP = "127.0.0.1"
localPort = 8080
secretKey = "your-secret-key"
```

**访问端（连接服务）：**
```toml
[[visitors]]
name = "visitor-web"
type = "stcp"
serverName = "home-pc.secret-web"  # 格式：user.name
secretKey = "your-secret-key"
bindAddr = "127.0.0.1"
bindPort = 18080  # 本地访问 127.0.0.1:18080 即穿透到对端
```

### 常用 CLI 命令

```bash
# 服务端
./frps -c frps.toml                    # 前台运行
./frps -c frps.toml --token "xxx"      # 命令行覆盖配置

# 客户端  
./frpc -c frpc.toml                    # 前台运行
./frpc -c frpc.toml -s 12059           # 指定服务端端口（覆盖配置）
./frpc reload -c frpc.toml             # 热重载配置（不中断）
./frpc status -c frpc.toml             # 查看连接状态
```