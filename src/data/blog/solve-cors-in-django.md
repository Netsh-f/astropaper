---
author: Zhang Wenjin
pubDatetime: 2025-12-02T06:00:00.000Z
title: 解决Django后端中的CORS跨域问题
tags:
  - docs
  - Django
description: 出现在前后端分离的纯后端Django项目中，通过添加中间件、修改配置等方式解决
draft: false
hideEditPost: true
timezone: Asia/Shanghai
---

完成以下配置以解决跨域问题

## 添加APP

```python file="settings.py"
INSTALLED_APPS = [
    ...
    'corsheaders',
]
```

## 添加中间件

安装依赖

```bash
pip install django-cors-headers
```

添加配置，注意该行要放在`MIDDLEWARE`的第一行，这是有顺序的

```python file="settings.py"
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    ...
]
```

## 配置

```python file="settings.py"
CORS_ORIGIN_ALLOW_ALL = True

CORS_ALLOW_CREDENTIALS = True

CORS_ALLOW_METHODS = [
    'GET',
    'POST',
    'PUT',
    'DELETE',
    'OPTIONS',
]

CORS_ALLOW_HEADERS = ('*', 'x-csrftoken', 'authorization', 'content-type')
```
