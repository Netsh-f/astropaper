---
author: Zhang Wenjin
pubDatetime: 2026-04-17T06:00:00.000Z
title: harbor 教程
tags:
    - docs
    - tutorial
description: 安装与常用配置
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 安装

### 下载

参考[官网](https://goharbor.io/docs/2.14.0/install-config/)

在 github [release页面](https://github.com/goharbor/harbor/releases) 下载 offline 安装包`harbor-offline-installer-v2.15.0.tgz`

解压
```bash
tar zxvf harbor-offline-installer-v2.15.0.tgz
```

### 证书

如果不搞证书，就要在连接harbor的时候，将ip加入`/etc/docker/daemon.json`的`-insecure-registry`

生成CA证书，替换`harbor.cheesechise.top`为实际域名
```bash
mkdir -p /data/cert && cd /data/cert
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha256 -days 3650 \
 -key ca.key \
 -out ca.crt \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=harbor.cheesechise.top"
```

生成服务器证书，记得替换域名和ip
```bash
openssl genrsa -out harbor.cheesechise.top.key 4096
openssl req -sha256 -new \
    -key harbor.cheesechise.top.key \
    -out harbor.cheesechise.top.csr \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=harbor.cheesechise.top"
```

这个命令如果出现权限问题，请使用vim创建`v3.ext`文件
```bash
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor.cheesechise.top
IP.1=49.233.70.33
EOF
```
```bash
openssl x509 -req -sha256 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in harbor.cheesechise.top.csr \
    -out harbor.cheesechise.top.crt
```

回到解压目录，编辑`harbor.yml`
```bash
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```
修改
```yml file="harbor.yml"
hostname: harbor.cheesechise.top

https:
  port: 443
  certificate: /data/cert/harbor.cheesechise.top.crt
  private_key: /data/cert/harbor.cheesechise.top.key
```

让本地docker信任证书
```bash
sudo mkdir -p /etc/docker/certs.d/harbor.cheesechise.top/

sudo cp /data/cert/harbor.cheesechise.top.crt /etc/docker/certs.d/harbor.cheesechise.top/
sudo cp /data/cert/harbor.cheesechise.top.key /etc/docker/certs.d/harbor.cheesechise.top/
sudo cp /data/cert/ca.crt /etc/docker/certs.d/harbor.cheesechise.top/

sudo systemctl restart docker
```

### 执行安装脚本

在解压目录
```bash
sudo ./prepare
sudo ./install.sh
```

### 其它

记得打开服务器`443`HTTPS端口

如果其他机器的docker要连接这个harbor，也要执行`让docker信任证书`这一个步骤，将`ca.crt`拷贝到该机器的`/etc/docker/certs.d/yourdomain.com/`目录下
```bash
# 1. 创建目录
sudo mkdir -p /etc/docker/certs.d/harbor.cheesechise.top/

# 2. 只拷贝 ca.crt，并重命名为 domain.crt (或者直接叫 ca.crt 也可以)
# 你需要从 Harbor 服务器把 ca.crt 传到这台机器上
sudo cp ca.crt /etc/docker/certs.d/harbor.cheesechise.top/

# 3. 重启 Docker（或者通常直接登录即可）
sudo systemctl restart docker
```

或者使用系统级信任
```bash
cp ca.crt /usr/local/share/ca-certificates/harbor.crt
sudo update-ca-certificates
sudo systemctl restart docker
```