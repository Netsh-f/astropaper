---
author: Zhang Wenjin
pubDatetime: 2026-01-23T06:00:00.000Z
title: FastAPI 学习笔记
tags:
    - docs
description: 学习搭建一个 FastAPI 项目
draft: False
timezone: Asia/Shanghai
---

## 前言

这是我的学习笔记，也不算个教程

## 创建项目

以后再总结

## 启动项目

```bash
uvicorn main:app --host 0.0.0.0 --port 9200 --reload
```

## Lifespan

Lifespan 是用来执行一些需要在项目启动/关闭时，只执行一次的代码的。这在你需要初始化一些比较耗时的任务的时候很有用，比如加载一个机器学习模型

参考 [Lifespan](https://fastapi.tiangolo.com/advanced/events/#lifespan)

一个简单的例子

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    print("start")
    yield
    print("end")

app = FastAPI(lifespan=lifespan)
```

那么，会在启动项目的时候输出`start`，在关闭项目的时候输出`end`
