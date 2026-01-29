---
author: Zhang Wenjin
pubDatetime: 2026-01-29T06:00:00.000Z
title: Django 项目启动指南
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

即使一切正常，也别着急执行 python manage.py migrate，因为如果你需要自定义 User Model，这个操作需要在第一次迁移数据库之前做，否则你需要在创建了自定义 User Model 之后删掉数据库重新迁移。

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

## 创建 App

这里以创建一个名为`account`的 App 为例，我一般习惯把系统里的账户相关的功能都放在这里。

```bash
python manage.py startapp account
```

创建了 App 之后需要注册才会生效

```python file="projectname/settings.py"
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'account',  # 添加这一行，注册新创建的 App
]
```

## 自定义 User / Model 创建方法

Django 里面一个 Model 就对应一个数据库表，它有强大的 ORM 框架，你不需要自己去写 SQL。

Django 有自带的完善的用户系统，有默认的 User Model，但是如果你需要自定义用户模型（比如添加额外的字段），你需要创建一个继承自`AbstractUser`的自定义 User Model。

```python file="account/models.py"
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    realname = models.CharField(max_length=128)
    idcard = models.CharField(max_length=18)
    phone_number = models.CharField(max_length=32)
    # 你可以添加更多字段
```

指定自定义 User Model

```python file="projectname/settings.py"
AUTH_USER_MODEL = 'account.User'
```

写好了一个 Model 之后，就可以进行数据库迁移

```bash
python manage.py makemigrations
python manage.py migrate
```

## 写一个注册接口（view）

```python file="account/views.py"
from rest_framework.decorators import api_view
from django.contrib.auth import get_user_model
from utils.decorators import validate_request
from utils.response import response_util
from utils.constants import ErrorCode

from account.request_serializer import RegisterSerializer

User = get_user_model()

@api_view(["POST"])
@validate_request(RegisterSerializer)
def register(request, data):
    if User.objects.filter(username=data.get("username")):
        return response_util(ErrorCode.USERNAME_ALREADY_REGISTERED)
    User.objects.create_user(**data)
    return response_util(ErrorCode.SUCCESS)
```

这里面用到了我好多个包装好的工具，这是我长期实践总结出来的一些东西，下面一一列出来

### @api_view

首先作为一个纯后端项目，建议使用`@api_view`，它是 Django REST Framework (DRF) 提供的装饰器，相较于`@require_http_methods`要强大的多

你需要安装它

```bash
uv add djangorestframework
```

### Serializer 序列化器校验传入参数

我通过`rest_framework.serializers`序列化器和自己编写的`validate_request`装饰器，可以快速地校验传入参数的合规性，代码如下

```python file="account/request_serializer.py"
from rest_framework import serializers

class RegisterSerializer(serializers.Serializer):
    username = serializers.CharField(max_length=128)
    password = serializers.CharField(max_length=128)
```

这个序列化器功能强大，可以写很多复杂的校验逻辑

```python file="utils/decorators.py"
from functools import wraps


from utils.constants import ErrorCode
from utils.response import response_util


def validate_request(serializer_class):
    def decorator(view_func):
        @wraps(view_func)
        def wrapped_view(request, *args, **kwargs):
            if request.method == "GET":
                serializer = serializer_class(data=request.GET)
            else:
                serializer = serializer_class(data=request.data)
            if serializer.is_valid():
                return view_func(
                    request, data=serializer.validated_data, *args, **kwargs
                )
            else:
                return response_util(
                    ErrorCode.INVALID_PARAM, f"invalid data: {serializer.errors}"
                )

        return wrapped_view

    return decorator
```

这个自己写的装饰器可以快速使用上面的序列化器，并且把不管是POST还是GET的参数都存放到data参数中，方面使用

### 返回数据封装

```python file="utils/response.py"
from rest_framework.response import Response
from rest_framework import status
from utils.constants import ErrorCode


def response_util(errno: ErrorCode, data=None) -> Response:
    if data is None:
        data = {}
    return Response({"errno": errno.value, "data": data}, status=status.HTTP_200_OK)
```

```python file="utils/constants.py"
from enum import Enum


class ErrorCode(Enum):
    SUCCESS = 0
    #  其他错误代码
```

通过这样的封装，所有的接口的返回都是一样的格式

```json
{
    "errno": 0,
    "data": {}
}
```

## 注册路由

首先需要在app目录中创建一个`urls.py`

```python file="account/urls.py"
from django.urls import path

from account import views

urlpatterns = [
    path('register/', views.register),
]
```

然后再项目总`urls.py`中引入

```python file="projectname/urls.py"
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path("account/", include('account.urls')),
]
```

这样，就可以请求`http://127.0.0.1:8000/account/register/`

## 关于登录

登录接口编写

```python file="account/views.py"
from django.contrib.auth import login, authenticate

@api_view(['POST'])
@validate_request(LoginSerializer)
def login_view(request, data):
    user = authenticate(**data)
    if user is None:
        return response_util(ErrorCode.PASSWORD_ERROR)
    login(request, user)
    return response_util(ErrorCode.SUCCESS, UserSerializer(user).data)
```

这里用到了返回数据的序列化`UserSerializer`，这个也是自己写的，用来将model的数据转换成字典，方便返回。这个序列化器很强大，可以自定义一些需要计算的字段，可以排除某些字段等等。

```python file="account/serializer.py"
from rest_framework.serializers import ModelSerializer
from django.contrib.auth import get_user_model

User = get_user_model()


class UserSerializer(ModelSerializer):
    class Meta:
        model = User
        fields = ["id", "username"]
```

这个登录接口被调用后，会在`Response Cookie`中存储一个名为`csrftoken`的字段，这个字段是Django自带的一个CSRF防护机制，同时也是用户的token。

前端应该将其存储起来，调用其他需要登录的接口的时候，就需要将它放在请求头中，字段名为`X-CSRFToken`。

如果你在使用 Apifox 或者 Postman 这些工具测试接口，你应该给登录这个接口添加一个提取变量的后置操作，将`csrftoken`提取到环境变量中，然后设置一个全局参数，在请求头中添加`X-CSRFToken: {{csrftoken}}`。

如果你想强制清除登录状态，你可以清空 Cookie。

如果你要在其他接口校验当前请求的用户有没有登录，我写了如下修饰器

```python file="utils/decorators.py"
def login_required(view_func):
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            return response_util(ErrorCode.NOT_LOGGED_IN)
        return view_func(request, *args, **kwargs)

    return wrapper
```

把它放在需要登录的接口上，比如这个登出接口

```python file="account/views.py"
@api_view(['GET'])
@login_required
def logout_view(request):
    logout(request)
    return response_util(ErrorCode.SUCCESS)
```

我知道 Django 有一个自带的登录校验装饰器`@login_required`，但是它适合在前后端不分离项目中使用。它会强制重定向到登录页面，不能自定义返回内容，在前后端分离项目中并不合适。

## TODO

Model各种类型字段，序列化，序列化器内置函数，级联序列化，异步定时任务，文件系统，生产环境部署运维……