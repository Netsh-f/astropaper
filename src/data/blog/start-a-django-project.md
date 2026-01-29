---
author: Zhang Wenjin
pubDatetime: 2026-01-29T06:00:00.000Z
title: Django 项目 Quick Start
tags:
    - docs
    - django
description: 从 0 搭建一个 Django 后端项目
draft: False
timezone: Asia/Shanghai
---

## 创建项目

使用 PyCharm 选择 Django 项目模板创建

## 修改时区与语言

```python file="projectname/settings.py"
LANGUAGE_CODE = "zh-hans"
TIME_ZONE = "Asia/Shanghai"
```

## 配置数据库

以 `MySQL` 为例

为项目写一个配置文件`config.yaml`

```yaml file="config.yaml"
DATABASE:
  ENGINE: django.db.backends.mysql
  USER: username
  PASSWORD: password
  HOST: 192.168.0.123
  NAME: database_name
  PORT: 3306
  OPTIONS:
    charset: utf8mb4
```

在 `settings.py` 中引入配置文件

```python file="projectname/settings.py"
import yaml

with open("config.yaml", 'r') as f:
    CONFIG = yaml.safe_load(f)

DATABASES = {
    "default": CONFIG['DATABASE']
}
```

安装`mysqlclient`驱动

```bash
uv add mysqlclient
```

检查配置是否成功

```bash
python manage.py check
```

这个命令会检查整个项目的配置，包括数据库连接等。如果一切正常，它会显示`System check identified no issues (0 silenced).`

即使一切正常，也别着急执行 python manage.py migrate，后面会说。

> mysqlclient 是官方推荐的驱动，底层是C语言，效率较高，但是需要安装C依赖；如果遇到困难，可以使用`PyMySQL`驱动，它是纯Python实现的，能确保成功。

如果你使用`PyMySQL`驱动连接`MySQL`，你需要

```bash
uv add pymysql
```

然后添加代码

```python file="projectname/__init__.py"
import pymysql

pymysql.version_info = (1, 4, 3, "final", 0)
pymysql.install_as_MySQLdb()
```

## 