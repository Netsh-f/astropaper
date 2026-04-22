---
author: Zhang Wenjin
pubDatetime: 2026-03-02T06:00:00.000Z
title: Redis 安装与配置
tags:
  - redis
  - database
  - linux
description: windows / linux 下安装与基本配置
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 安装

参考 [菜鸟教程-redis](https://www.runoob.com/redis/redis-install.html)

### Windows

访问 [redis-windows](https://github.com/redis-windows/redis-windows/releases)

选择`Redis-x.x.x-Windows-x64-msys2.zip`

解压到任意目录，运行`start.bat`

### Linux

对于 ubuntu 系统，直接 apt

```bash
sudo apt update
sudo apt install redis-server
```

## 配置

```bash
vim /etc/redis/redis.conf
```

主要修改以下几个选项

默认配置

```conf file="/etc/redis/redis.conf"
bind 127.0.0.1 ::1
protected-mode yes
# requirepass foobared
```

修改后的配置

```conf file="/etc/redis/redis.conf"
bind 0.0.0.0
protected-mode no
requirepass yourpassword
```

## 运行

```bash
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

检查运行状态
```bash
sudo systemctl status redis-server
redis-cli ping
```
