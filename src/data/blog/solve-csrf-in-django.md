---
author: Zhang Wenjin
pubDatetime: 2025-12-03T06:00:00.000Z
title: 如何通过Django后端CSRF校验
tags:
  - django
  - docs
  - tutorial
description: 如何获取、传入csrf-token，指定前端部署地址
draft: false
hideEditPost: true
timezone: Asia/Shanghai
---

## 获取csrftoken

一个一般的登录接口是这样写的

```python file="account/views.py"
from django.contrib.auth import login, authenticate

@api_view(['POST'])  
@validate_request(LoginSerializer)  
def login_view(request, data):  
    user = authenticate(**data)  
    if user is None:  
        return response_util(ErrorCode.PASSWORD_ERROR)  
    login(request, user)  
    return response_util(ErrorCode.SUCCESS, CustomUserSerializer(user).data)
```

当前端或一些接口调用工具（Postman、Apifox）请求该接口并登录成功后，`Response Cookie`中会带有一个 Cookie 名为`csrftoken`的字段。前端应该将其存储起来，接口调用工具可以使用接口后置操作提取该变量。

我不太懂前端，有些和我前端对接的同学，会要求我设置

```python file="settings.py"
SESSION_COOKIE_HTTPONLY = False 
```

否则他们拿不到 Cookie，也就拿不到 csrftoken。但我查了一下，Session Cookie 和 CSRF Cookie 是两个不同的 Cookie，csrftoken 是可以直接从`document.cookie`中获取到的。总之如果前端拿不到 Cookie，你可以尝试设置一下这个字段，请注意它会带来跨站脚本攻击（XSS）风险。
## 传入并校验csrftoken

在请求其他接口时，应在请求的 Header 中添加名为`X-CSRFToken`的参数，其值为上一步获取的`csrftoken`

Django 后端在接受到该请求后，会自动校验该 token。若通过则`request.user.is_authenticated`为`true`，可以通过这种方法，判断用户是否登录。

例如下面是一个校验用户已登录的修饰器，可以写在任何需要登录的接口前

```python file="login_required.py"
from functools import wraps

def login_required(view_func):  
    @wraps(view_func)  
    def wrapper(request, *args, **kwargs):  
        if not request.user.is_authenticated:  
            return response_util(ErrorCode.NOT_LOGGED_IN)  
        return view_func(request, *args, **kwargs)  
    return wrapper
```

## 配置前端部署地址

无论前端是部署在某个服务器上还是你的同事在本地电脑测试，都需要将对应 IP 和端口配置在 Django 项目中。这项要求似乎是最近（24-25年） Django 某次更新后新增的。配置方法如下

```python file="settings.py"
CSRF_TRUSTED_ORIGINS = [  
    'http://localhost:5173',  
    'http://127.0.0.1:5173',  
    'http://10.61.1.201:5173',  
    'http://10.70.251.43:9167',  
    'http://10.70.251.43:9163',  
]
```