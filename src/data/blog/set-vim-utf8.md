---
author: Zhang Wenjin
pubDatetime: 2026-01-15T02:00:00.000Z
title: 设置Vim默认使用UTF-8解决乱码
tags:
    - tutorial
    - docs
    - linux
description: 修改.vimrc
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 方法

编辑`~/.vimrc`文件（如果没有则自行创建），添加如下内容

```bash src="~/.vimrc"
set encoding=utf-8
set fileencodings=utf-8,gbk,latin1
set termencoding=utf-8
```

encoding：Vim 内部使用的编码

fileencodings：Vim 自动探测文件编码的顺序（优先尝试 utf-8）

termencoding：终端使用的编码（现代终端基本都是 UTF-8）
