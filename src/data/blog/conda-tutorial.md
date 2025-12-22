---
author: Zhang Wenjin
pubDatetime: 2023-05-20T06:00:00.000Z
title: Conda快速入门教程
tags:
    - docs
    - tutorial
description: 安装方法与常用命令
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 安装

下载安装脚本并执行

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

安装在`~/miniconda3`就可以

安装完成后，需要执行初始化命令

```bash
~/miniconda3/bin/conda init
```

然后重启终端，安装完成

## 常用命令

### 创建新的环境

```bash
conda create --name MyEnv python=3.9
```

### 启用虚拟环境

```bash
conda activate MyEnv
```

### 退出虚拟环境

```bash
conda deactivate
```

### 列出所有虚拟环境

```bash
conda env list
```

### 删除一个虚拟环境

```bash
conda remove --name MyEnv --all
```

执行上面命令虽然删除了环境内容，但并不会删除环境目录，这意味着你无法再创建一个同名环境。你需要到`~/miniconda3/envs/`目录中手动删除环境目录。