---
author: Zhang Wenjin
pubDatetime: 2026-04-16T06:00:00.000Z
title: opencode 教程
tags:
    - docs
    - tutorial
description: 安装与常用配置/快捷键
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 安装

参考[官网](https://opencode.ai/)

```bash
curl -fsSL https://opencode.ai/install | bash
```

如果是Windows系统，请使用wsl安装

## 配置

在目录`~/.config/opencode/`中，可以配置

* agents
* skills
* tools

## TUI 快捷键

`Ctrl + P` 显示所有命令

`/help` 帮助

`/connect` 添加提供商

`/editor` 使用编辑器编辑对话消息，需要在环境变量中设置编辑器，配置`vim`或`vscode`如下

```bash file="~/.bashrc"
export EDITOR=vim
export EDITOR="code --wait"
```

`/models` 列出可用模型，切换模型

其他命令详见[官方文档](https://opencode.ai/docs/zh-cn/tui/#_top)