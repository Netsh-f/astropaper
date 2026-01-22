---
author: Zhang Wenjin
pubDatetime: 2026-01-22T06:00:00.000Z
title: uv 使用教程
tags:
    - docs
description: uv 安装、使用、cuda torch安装、换源等
draft: False
timezone: Asia/Shanghai
---

## 前言

conda似乎已经被嫌弃了，合作的企业都开始用uv了，我也应该跟着转型

## 安装

参考[uv installation](https://docs.astral.sh/uv/getting-started/installation/)

## 基本使用

创建虚拟环境，需要在项目根目录下执行，它会创建一个`.venv`

```bash
uv venv --python 3.10.19
```

启用创建的环境，同样在项目根目录下

```bash
source .venv/bin/activate
```

在项目已有`pyproject.toml`和`uv.lock`的情况下，同步项目依赖

```bash
uv sync
```

## 安装 GPU 版本的 PyTorch

参考[Using uv with PyTorch](https://docs.astral.sh/uv/guides/integration/pytorch/#installing-pytorch)

特定 CUDA 版本的 PyTorch 不在标准 PyPI 包上面，如果你直接使用 uv 安装，会安装到 cpu 版本。

正确的做法参考官方文档

首先修改`pyproject.toml`，以`cu118`为例

```toml src="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14"
dependencies = [
  "torch>=2.9.1",
  "torchvision>=0.24.1",
]

[[tool.uv.index]]
name = "pytorch-cu118"
url = "https://download.pytorch.org/whl/cu118"
explicit = true
```

然后执行

```bash
uv lock
```

这一步会更新`uv.lock`文件

最后执行

```bash
uv sync
```

## 换源

uv 默认从 github 下载预编译的 python 二进制，所以创建虚拟环境的时候可能会卡住，需要换源

```bash
export UV_PYTHON_INSTALL_MIRROR="https://python-standalone.org/mirror/astral-sh/python-build-standalone"
```

清华源

```bash
export UV_PYTHON_DOWNLOADS_URL=https://mirrors.tuna.tsinghua.edu.cn/github-release/astral-sh/python-build-standalone/
```

> 试了试下清华源，不咋快

参考[知乎-用uv管理python环境(各种应用场景)-VCVC](https://zhuanlan.zhihu.com/p/1908630076704159010)