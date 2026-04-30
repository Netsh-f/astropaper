---
author: Zhang Wenjin
pubDatetime: 2026-04-30T06:00:00.000Z
title: Cloc 统计项目代码行数工具
tags:
  - cloc
description: 安装与使用
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

cloc 可以快速统计目录中代码的行数信息，比statistic好用

## 安装

Windows 进入 [Github Release](https://github.com/AlDanial/cloc/releases)页面，下载`cloc-x.xx.exe`，放在本地并添加环境变量即可使用

Linux 使用 apt 安装
```bash
sudo apt update
sudo apt install cloc
```

## 使用

```bash
cloc .
cloc . --exclude-dir=node_modules,dist
cloc . --include-lang=Python,Java
cloc . --by-file
cloc . --out=result.txt
cloc . --exclude-ext=log,lock
cloc . --not-match-f="config\.js"  # 正则表达式
```