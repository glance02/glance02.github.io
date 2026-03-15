---
layout:     post
title:      "n2n Supernode 部署"
subtitle:   "在 Linux 服务器上编译并部署 n2n 组网节点"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 网络服务
---
## 获取并编译 n2n

1. 安装编译依赖
```bash
apt update
apt install -y build-essential autoconf automake libtool pkg-config git
```
2. 下载源码并生成 configure
这一部分我感觉更重要的还是看作者自己的编译的readme。
```bash
git clone https://github.com/ntop/n2n.git
cd n2n
./autogen.sh
```
> `autogen.sh` 用于生成 `configure` 脚本（源码仓库默认不带）
3. 编译并安装
```bash
./configure
make

# optionally install
make install
```
### 配置 systemd

1. 创建 systemd 服务文件
```bash
vim /etc/systemd/system/supernode.service
```
	最终我正确的服务文件：
```service
[Unit]
Description=n2n Supernode
After=network.target

[Service]
Type=simple
ExecStart=/usr/sbin/supernode -f -p 9527
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```
关键说明：
- `-f`：禁止 supernode 自行 daemonize（systemd 必须）
- `-p`：supernode 主监听端口
- `Type=simple`：与前台进程模型匹配
2. 启动并设置开机自启
```bash
systemctl daemon-reload
systemctl start supernode
systemctl enable supernode
```
### 验证运行状态

1. 查看服务状态
```bash
systemctl status supernode
```
2. 确认端口监听
```bash
ss -ulnp | grep supernode
```
### 防火墙与安全组

自己关一下服务器防火墙，或者选择性放开一些。最重要的是需要在服务器的安全组部分放开端口。需要放开的端口有：
- UDP 9527
- TCP 9527