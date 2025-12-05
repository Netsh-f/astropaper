---
author: Zhang Wenjin
pubDatetime: 2024-07-18T06:00:00.000Z
title: Linux系统下压缩相关命令
tags:
    - docs
    - linux
description: tar, tar.gz, zip等压缩格式的解压缩命令
draft: False
timezone: Asia/Shanghai
---

## .tar

压缩

```bash
tar -cvf archive.tar file1 file2 directory
```

* `-c`压缩
* `-v`显示输出
* `-f`指定输出文件名称

解压

```bash
tar -xvf archive.tar
```

* `-x`解压

列出压缩文件内容

```bash
tar -tvf archive.tar
```

* `-t`列出内容

## .tar.gz

相较于`.tar`加`-z`参数

压缩

```bash
tar -zcvf archive.tar.gz directory
```

解压

```bash
tar -zxvf example.tar.gz
```

## 其他压缩格式

和`.gz`一样，只需替换`-z`参数为其他参数

* `.tar.bz2` 使用 `-j` 参数
* `.tar.xz` 使用 `-J` 参数

## .zip

安装包

```bash
apt install zip unzip
```

压缩

```bash
zip -r archive.zip directory
```

* 可以添加`-q`参数，不显示压缩过程输出

解压

```bash
unzip archive.zip -d directory
```

* `-d`参数指定解压目录

查看文件内容

```bash
unzip -l archive.zip
```