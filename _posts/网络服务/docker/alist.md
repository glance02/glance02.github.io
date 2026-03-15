---
layout:     post
title:      "Docker 部署 Alist"
subtitle:   "离线拉取镜像后通过 SFTP 上传至服务器部署 Alist"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 网络服务
    - Docker
---
服务器网络不稳定，`docker pull` / `curl` 经常失败。如果是普通的访问外网可以使用ssh反向代理，但是docker似乎有点不一样，反向代理还是无法正常拉取。多次尝试觉得本地拉取再用sftp上传是最简单的办法。

## 本地机器操作

1. 拉取镜像。
```bash
docker pull xhofe/alist:latest
```
2. 打包镜像
```bash
docker save xhofe/alist:latest -o alist.tar
```
3. 上传到服务器。自己用termius或者其他工具上传。

## 服务器端操作

1. 导入镜像
```bash
docker load -i alist.tar
```
	可以再确认一下镜像是否存在：
```bash
docker images | grep alist
```
2. 写`docker-compose.yaml`文件
	这次 alist的compose文件如下：
```yaml
services:
   alist:
     image: xhofe/alist:latest
     platform: linux/amd64
     container_name: alist
     volumes:
       - /etc/alist:/opt/alist/data
     ports:
       - "5244:5244"
     restart: unless-stopped
```
3. 启动
```bash
docker compose up -d
```
### 初始化与管理

1. 设置管理员密码
```bash
docker exec -it alist ./alist admin set YOUR_PASSWORD
```
说明：
- 密码只在**首次启动**自动生成并输出
- 后续只能通过 `admin set` 或 `admin random` 修改
2. 进入容器
```bash
docker exec -it alist bash
```
可以在这里进去alist容器，但是不建议在这里创建一些需要的文件夹。
### 文件与目录说明

1. 数据持久化
```text
宿主机：/etc/alist
容器内：/opt/alist/data
```
所有配置 / 数据库 / 存储信息都在宿主机。重建容器不会丢数据。
2. 创建文件夹的正确方式
本地存储（Local），在宿主机 `/etc/alist/...` 下创建。不要依赖容器内部目录。容器删除即丢失。

### 创建存储目录参数说明

只在此说个别参数。注意alist里面创建的都是从容器内部的视角来看的地址。
1. **显示文件夹名称**：这个只是如其名一般的显示名称，在alist的网站显示的文件夹的名称。这个名称和webdav的设置也是有关的。一般是````http://[ip地址]:5244/dav/显示文件夹名称/````
2. **根文件夹路径**：这个需要看容器内部的地址。和yaml文件有关。我的是在`/opt/alist/data/`以下的文件夹。注意下一级填写的文件夹需要存在，所以可能需要先在映射的文件夹下创建。

