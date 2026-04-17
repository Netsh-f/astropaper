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

建议先跳过次步骤，直接使用HTTP

安装`certbot`
```bash
sudo apt update
sudo apt install certbot -y
```

申请证书
```bash
sudo certbot certonly --standalone -d harbor.cheesechise.top
```
执行此步骤要确保80端口没有被占用，输入邮箱并同意服务条款

将生成的证书配置给Harbor

```yml file="harbor.yml"
https:
  port: 443
  # 使用 Let's Encrypt 的全链证书
  certificate: /etc/letsencrypt/live/harbor.cheesechise.top/fullchain.pem
  # 使用 Let's Encrypt 的私钥
  private_key: /etc/letsencrypt/live/harbor.cheesechise.top/privkey.pem
```

### 执行安装脚本

在解压目录
```bash
sudo ./prepare
sudo ./install.sh
```

## nginx 端口转发

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl status nginx
sudo vim /etc/nginx/sites-available/harbor.conf
```

填入如下配置

```conf file="harbor.conf"
server {
    listen 80;
    server_name harbor.cheesechise.top;
    # 将 HTTP 自动跳转到 HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name harbor.cheesechise.top;

    # 指向你刚刚申请成功的 Certbot 证书
    ssl_certificate /etc/letsencrypt/live/harbor.cheesechise.top/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/harbor.cheesechise.top/privkey.pem;

    # 必须加上这一行，否则推送镜像时会报 413 错误（上传文件太大）
    client_max_body_size 0;

    location / {
        # 转发到你的 Harbor 实际运行端口
        proxy_pass https://127.0.0.1:9181;
        
        # 保持连接的必要请求头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 禁用缓存以支持 Docker 推送大文件流
        proxy_buffering off;
        proxy_request_buffering off;
    }
}
```

激活配置
```bash
sudo ln -s /etc/nginx/sites-available/harbor.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 自己生成证书（不要使用此方法，很麻烦）

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
修改，可以修改端口号
```yml file="harbor.yml"
hostname: harbor.cheesechise.top

http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

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