---
author: Zhang Wenjin
pubDatetime: 2026-03-08T06:00:00.000Z
title: 如何搭建 MinIO 服务
tags:
  - minio
  - object-storage
  - docker
description: 使用 Docker 部署的命令
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

MinIO是一个文件服务器

部署命令

```bash
docker run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -v ~/minio/data:/data \
   -e "MINIO_ROOT_USER=admin" \
   -e "MINIO_ROOT_PASSWORD=YourPassword" \
   -d --restart=always \
   quay.io/minio/minio server /data --console-address ":9001"
```
