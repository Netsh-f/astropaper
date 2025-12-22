---
author: Zhang Wenjin
pubDatetime: 2025-09-29T06:00:00.000Z
title: 如何设置Huggingface镜像
tags:
    - docs
    - tutorial
description: 添加镜像与token环境变量的方法
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

在`from transformers`语句或其他需要 Huggingface 的库之前添加

```python
import os
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"
os.environ["HF_HUB_ENABLE_HF_MIRROR"] = "true"
os.environ['HF_TOKEN'] = "hf_Your_token"
```

其中，token从[这里](https://huggingface.co/settings/tokens)获取

当然你也可以直接在系统环境变量中设置