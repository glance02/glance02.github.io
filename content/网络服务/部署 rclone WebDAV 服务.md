
## 原因

原本使用 AList 挂载本地存储并对外提供 WebDAV 服务，供 Zotero 和 Obsidian 同步使用。排查后发现 AList 的 WebDAV 实现不完整，`DELETE` 方法返回 405，导致 Zotero 和 Obsidian 验证服务器时失败。rclone 的 WebDAV 实现完整，支持所有必要的方法，因此用它替代 AList 提供 WebDAV 服务。

---

## 部署流程

**1. 安装 rclone**

```bash
apt update && apt install rclone -y
```

**2. 创建存储目录**

```bash
mkdir -p /opt/webdav/zotero
mkdir -p /opt/webdav/obsidian
```

**3. 创建 systemd 服务文件**

```bash
nano /etc/systemd/system/rclone-webdav.service
```

```ini
[Unit]
Description=rclone WebDAV Server
After=network.target

[Service]
Type=simple
ExecStart=rclone serve webdav /opt/webdav --addr 127.0.0.1:6080 --user admin --pass 你的密码 --vfs-cache-mode full --log-level INFO
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**4. 启动服务并设置开机自启**

```shell
systemctl daemon-reload
systemctl enable rclone-webdav
systemctl start rclone-webdav
```

**5. 配置 Nginx 反代**

```bash
nano /etc/nginx/conf.d/webdav.conf
```

```nginx
server {
    listen 80;
    server_name webdav.glance02.xyz;
    client_max_body_size 500m;

    location / {
        proxy_pass http://127.0.0.1:6080;
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

```bash
nginx -t && systemctl reload nginx
```

**6. 客户端配置**

| 应用       | URL                                    |
| -------- | -------------------------------------- |
| Zotero   | `http://webdav.glance02.xyz/zotero/`   |
| Obsidian | `http://webdav.glance02.xyz/obsidian/` |

用户名和密码填服务文件里设置的账号密码。

