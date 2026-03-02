---
author: Zhang Wenjin
pubDatetime: 2026-03-02T06:00:00.000Z
title: MySQL 安装与配置
tags:
    - docs
    - tutorial
description: linux 下安装与基本配置
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 安装

ubuntu 系统中运行

```bash
sudo apt update
sudo apt install mysql-server
```

然后运行

```bash
sudo mysql_secure_installation
```

这个命令会引导你设置安全配置，新版 mysql 在本机使用 root 权限登录的时候，会使用 auth_socket 插件进行身份认证，登陆到 mysql root 账户不需要密码，所以不用设置 root 密码

## 创建一个新用户

```bash
mysql -u root
create user 'newusername'@'localhost' identified by 'password';
```

如果希望在任意地方都可以使用这个用户访问到数据库，那么将localhost改为通配符%

给用户授权

```bash
grant all privileges on databasename.tablename to 'newusername'@'localhost';
```

如果希望授权所有表的权限，则将`databasename.tablename`改为`*.*`，localhost改为%条件同上

刷新用户权限并退出

```bash
flush privileges;
exit;
```