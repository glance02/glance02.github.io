---
layout:     post
title:      "Docker 部署私有 DERP 节点"
subtitle:   "基于 ip_derper 搭建带客户端验证的私有 Tailscale DERP 服务器"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 网络服务
    - Docker
---
这个docker镜像会自己在镜像里面创建自签名证书。为了一定程度上保证我的derp的安全，可以添加客户端验证。

## 前置准备

1. **拉取镜像**：和在[alist](alist.md)中一样，先在本地pull，然后save上传到服务器，最后在服务器上load。
2. **下载tailscale**：按照如下指令：
	```bash
	curl -fsSL https://tailscale.com/install.sh | sh
	sudo tailscale up
	```
		之后会跳出认证提示，完成登录即可。
3. 开放端口。需要开放两个端口，一个tcp，一个udp，按照yaml文件中来即可。
## 创建并启动容器

创建`docker-compose.yaml`文件。
```yaml
services:
  derper:
    image: ghcr.io/yangchuansheng/ip_derper:latest
    container_name: derper
    restart: always
    ports:
      - "11223:11223"
      - "3478:3478/udp"
    environment:
      - DERP_ADDR=:11223
      - DERP_VERIFY_CLIENTS=true  # 初始部署关闭验证
    volumes:
  	  - /var/run/tailscale/tailscaled.sock:/var/run/tailscale/tailscaled.sock
```
启动服务：
```bash
docker compose up -d
```
可以输入`docker logs derper`来查看docker的运行信息。

## Tailscale控制台配置

1. **添加自定义DERP映射**
	登录 [Tailscale Admin Console](https://login.tailscale.com/admin/)，进入你的网络。在 **Access Controls** 页面，找到 `derpMap` 字段，添加配置：
	```json
	"derpMap": {
   		"OmitDefaultRegions": false, // 初始阶段设为false，与官方节点共存以便测试
   		"Regions": {
     	"901": {
       	"RegionID": 901,
       	"RegionCode": "my-derp",
       	"RegionName": "My Server",
       	"Nodes": [{
         		"Name": "1",
         		"RegionID": 901,
         		"HostName": "121.43.249.54", 
         		"DERPPort": 11223,
         		"STUNPort": 3478,
         		"InsecureForTests": true // 关键：允许使用自签名/不匹配证书
       		}]
     		}
   		}
 	}
	```
	最后记得点击save

## 验证

在客户端输入：
```bash
tailscale netcheck
```
检查是否出现自己的derp中继服务。