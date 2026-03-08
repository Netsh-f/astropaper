---
author: Zhang Wenjin
pubDatetime: 2026-02-10T06:00:00.000Z
title: 如何搭建 TeamSpeak 语音服务器
tags:
    - docs
    - tutorial
description: 官方方法及使用 Docker 快速搭建
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

Teamspeak(ts) 相较于 kook，占用内存小，对游戏体验影响小，和用于有固定游戏伙伴的小团体使用

本文章将记录如何搭建私有 TeamSpeak3 服务器

## 官方渠道

[官方网站](https://www.teamspeak.com/en/downloads/#server)

## Docker 快速部署

```bash
mkdir -p /data/teamspeak
chown -R 503:503 /data/teamspeak
docker run -d --restart=always --name teamspeak \
  -e PUID=503 \
  -e PGID=503 \
  -e TS3SERVER_GDPR_SAVE=false \
  -e TS3SERVER_LICENSE=accept \
  -p 9987:9987/udp -p 30033:30033 -p 10011:10011 -p 41144:41144 \
  -v /data/teamspeak:/data \
  mbentley/teamspeak
```

需要为服务器开放两个端口：
* `9987`: 语音端口，UDP协议
* `30033`: 文件传输端口，TCP协议

查看管理员密钥

```bash
docker logs teamspeak
```

会看到

```
------------------------------------------------------------------
                      I M P O R T A N T                           
------------------------------------------------------------------
      ServerAdmin privilege key created, please use it to gain 
      serveradmin rights for your virtualserver. please
      also check the doc/privilegekey_guide.txt for details.

       token=BWZ0R9FSmrOYxxxxxxxxxxxxTfwx0iFWsONRH
------------------------------------------------------------------
```

然后你作为第一个链接服务器的人，会成为管理员，它会要求输入密钥，就是上面的token