---
layout:     post
title:      "SSH 使用技巧"
subtitle:   "SSH 反向代理与服务器公钥配置"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 网络服务

---
## 使用-R实现反向代理
注意需要先打开clash的Alow Lan，即局域网连接。
1. 在安装 clash 的本地电脑上运行：
```
ssh -R 7897:localhost:7897 -p 6419 root@121.43.249.54
```
2. 在服务器上配置代理：
	```
	export https_proxy=http://127.0.0.1:7897
	export http_proxy=http://127.0.0.1:7897
	```
## 给远程服务器添加公钥

1. 复制自己的公钥内容
2. **登录到远程服务器执行：**
```bash
# 创建 .ssh 目录（如果不存在）
mkdir -p ~/.ssh

# 写入公钥
echo "粘贴你复制的公钥内容" >> ~/.ssh/authorized_keys

# 设置权限（关键！）
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
3. 重启ssh服务
```bash
sudo systemctl daemon-reload
sudo systemctl restart ssh
```