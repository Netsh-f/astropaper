---
author: Zhang Wenjin
pubDatetime: 2024-04-18T06:00:00.000Z
title: 生成ssh公钥并添加到Github/远程服务器
tags:
    - tutorial
    - docs
description: ssh-keygen命令生成ssh公钥
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 生成公钥

首先在你的开发机器上（以windows系统为例），在任意地方打开git bash，执行命令

```bash
ssh-keygen -t rsa -C "youremail@xxx.com"
```

其中`-t`参数执行 key 的类型，`-C`参数为该公钥的注释

执行之后一般情况下直接按三次回车使用默认值就可以了，它会在你的用户文件夹下的`.ssh/`文件夹下生成两个文件

`id_rsa`和`id_rsa.pub`

前者存储的是私钥，后者存储的是公钥。

复制`id_rsa.pub`文件的文本内容，一般形如

```src="id_rsa.pub"
ssh-rsa AAAxxxxxxxxxxxxxxxxxxxx= youremail@xxx.com
```

## 添加到Github

进入github点击右上角你的头像进入设置（settings）

左侧进入`SSH and GPG keys`标签页，点击`New SSH key`把刚刚生成的公钥复制进去并确认

之后你就可以使用ssh的方式访问远端仓库了

## 添加到远程服务器

将公钥添加到

```bash src="~/.ssh/authorized_keys"
ssh-rsa AAAxxxxxxxxxxxxxxxxxxxx= youremail@xxx.com
```

这样在使用ssh连接的时候就不需要每次都输入密码了