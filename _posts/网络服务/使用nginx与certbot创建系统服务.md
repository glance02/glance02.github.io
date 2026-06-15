
创建一个在本地某一个端口运行的服务，这次以创建一个 [[部署 rclone WebDAV 服务|rclone WebDAV 服务]]为例。

## 核心架构原理

通过将 Rclone 锁定在本地 `127.0.0.1`，所有公网流量、安全加密（SSL）均由 Nginx 和 Certbot 统一管理，确保系统安全与高效。

## 创建 Rclone 本地系统服务 (Systemd)

为新文件夹独立运行一个 Rclone 实例，仅监听本地端口。

**1. 创建服务文件：**

```Bash
vim /etc/systemd/system/rclone-limm.service
```

**2. 写入配置内容：**
```TOML
[Unit]
Description=rclone WebDAV Server (Limm)
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone serve webdav /opt/limm --addr 127.0.0.1:6081 --user limm --pass 0000t --vfs-cache-mode full --log-level INFO
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**3. 启动并设置开机自启：**

```Bash
sudo systemctl daemon-reload
sudo systemctl enable rclone-limm
```

## 配置 Nginx 反向代理 (HTTP 80)

在 Nginx 中为新的子域名建立规则，将外部请求转发至本地 Rclone 端口。

**1. 新建 Nginx 配置文件：**

```Bash
vim /etc/nginx/conf.d/limm.conf
```

**2. 写入基础转发配置（先配 80 端口，方便 Certbot 验证）：**

```Nginx
server {
    server_name limm.glance02.xyz;
    client_max_body_size 500m;

    location / {
        proxy_pass http://127.0.0.1:6081;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";
        proxy_http_version 1.1;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }

}
```

**3. 启用配置并重启 Nginx：**

```Bash
sudo nginx -t  # 检查语法是否正确
sudo systemctl restart nginx
```

## 使用 Certbot 自动部署 HTTPS 证书

利用 Certbot 为新域名申请 SSL 证书，并让其自动将 Nginx 的 80 端口配置升级为 443 端口（HTTPS）。

**1. 执行证书申请命令：**

```Bash
sudo certbot --nginx -d limm.glance02.xyz
```

**2. 流程说明：**

- Certbot 会自动与 Let's Encrypt 通信完成域名所有权验证。
    
- 验证成功后，Certbot 会**自动修改** `/etc/nginx/conf.d/limm.conf` 文件，添加 SSL 证书路径，并将 HTTP 重定向到 HTTPS。
    
- 证书到期前，系统内的 Certbot 定时任务会自动刷新该证书，无需手动维护。

## 第四步：常用运维命令

- **查看 Rclone 服务运行状态：**

    ``` Bash
    systemctl status rclone-limm
    ```
    
- **重启 Rclone / Nginx 服务：**
    
    ```Bash
    sudo systemctl restart rclone-limm
    sudo systemctl restart nginx
    ```

## 相关笔记

- [[部署 rclone WebDAV 服务]]
- [[alist]]
