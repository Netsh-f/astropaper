---
author: Zhang Wenjin
pubDatetime: 2024-03-31T06:00:00.000Z
title: Docker 安装及使用
tags:
  - docker
  - container
  - linux
description: 安装、配置、使用命令
draft: false
hideEditPost: false
timezone: Asia/Shanghai
---

## 安装教程

参考 [Docker 从入门到实践](https://yeasy.gitbook.io/docker_practice/di-yi-bu-fen-ru-men-pian/03_install/3.1_ubuntu)

## 镜像管理

构建镜像
```bash
docker build -t my-image:v1.0 .
docker build -t my_new_image_from_dockerfile -f ./Dockerfile .
```
将一个容器打包为一个新的镜像
```bash
docker commit <container_id> <new_image_name>
```
查看现有镜像
```bash
docker images
docker image ls
```
从仓库中拉取镜像
```bash
docker image pull <image_name>
```
一般一个完整的镜像名`<image_name>`的组成为
```
{image registry地址}/{仓库名}/{镜像名}:TAG
```
删除一个镜像
```bash
docker image rm <image_id>
```
`<image id>`可以使用`docker image ls`命令查看

为一个镜像设置别名
```bash
docker tag <image_name> <image_nickname>
```
但它们的image id是一样的，它们指向同一个image实体

这个功能在需要push一个镜像的时候比较有用。比如说你现在创建了一个镜像名为my_image现在想要将它推到一个私有仓库上，就可以使用tag功能将其命名为完整的镜像名scs.buaa.edu.cn:8081/yourproject/my_image:v1.0然后执行命令
```bash
docker push scs.buaa.edu.cn:8081/yourproject/my_image:v1.0
```

## 容器管理

容器有6个状态，执行边上的相应命令就可以使容器在各状态之间切换，参考[States of a Docker Container](https://www.baeldung.com/ops/docker-container-states)

![container_status](@/assets/images/docker_container_status.png)

运行一个镜像
```bash
docker run -it --rm --name container_name -v /path/in/host/:/path/to/container/ -p 8899:80 image_name:tag bash
```

* `-it`参数意为启动一个可交互的窗口
* `-rm`参数意为在当容器退出后会自动删除该容器，不会进入Exited状态
* `-v`参数可以让容器和宿主机共享目录`docker run -v <宿主机的目录>:<容器的目录> <image_name>`也可以执行数据卷，详见本篇博客下面内容
* `--name`参数指定容器的名字
* `-p`参数指定容器的某个端口暴露到宿主机的某个端口`-p <宿主机端口>:<容器端口>`
* `-d`参数，让容器运行在后台

删除所有标记为<none>的镜像
```bash
docker rmi $(docker images -f "dangling=true" -q)
```
查看正在运行的容器
```bash
docker ps -a
```
`-a`参数可以列出不活跃的（Exited）容器

查看一个容器的日志
```bash
docker logs <container_id>
```
回到一个正在运行的容器(进入它的bash shell)
```bash
docker exec -it continerID bash
```
结束容器，在容器内执行
```bash
exit
```
短暂退出容器会话
```
Ctrl + P + Q
```
回到容器会话
```
docker attach <container_id>
docker exec -it <container_id> bash
```

## 数据卷的使用

当需要持久化存储数据的时候，可以使用`docker volume`来存储数据

创建数据卷
```bash
docker volume create <volume_name>
```
在运行容器的时候挂载数据卷
```bash
docker run -v <volume_name>:/path/in/container/ <image_name>
```
查看已有数据卷
```bash
docker volume ls
```
查看单个数据卷信息
```bash
docker volume inspect <volume_name>
```
删除数据卷
```bash
docker volume rm <volume_name>
```
映射文件夹
```bash
docker run -v <宿主机的目录>:<容器的目录> <image_name>
```

## Docker仓库

登录私有仓库
```bash
docker login <私有仓库地址>
```
PUSH镜像

如果要push的镜像的名字不是完整的带有仓库地址，项目名的名字，需要先使用tag命令起一个别名，然后使用别名进行push。
```bash
docker tag myimage:tag <私有仓库地址>/projectname/imagename:tag
docker push <私有仓库地址>/projectname/imagename:tag
```
PULL镜像
```bash
docker pull <私有仓库地址>/projectname/imagename:tag
```

## 设置代理

### 给 docker 本身设置代理

对于Windows上的Docker Desktop可以打开Docker Desktop->Settings->Docker Engine或者在C:\Users\username\.docker\daemon.json中修改配置文件，添加下文"proxies"一项

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "insecure-registries": [
    "121.89.178.134:10378"
  ],
  "proxies": {
    "http-proxy": "http://10.192.106.144:7890",
    "https-proxy": "http://10.192.106.144:7890"
  }
}
```

其中如果你的代理服务在你的宿主机上，那么代理ip应该填写宿主机的内网ip而不是`127.0.0.1`，对于Windows可以通过`ipconfig`命令查看，对于Linux可以通过`ifconfig`命令查看

对于Linux系统，可以创建`daemon.json`
```json file="daemon.json"
{
  "proxies": {
    "http-proxy": "http://172.31.0.152:7890",
    "https-proxy": "http://172.31.0.152:7890",
    "no-proxy": "127.0.0.0/8"
  }
}
```

然后执行
```
cp daemon.json /etc/docker/
sudo systemctl restart docker
```

更多相关设置见官方文件[Daemon proxy configuration](https://docs.docker.com/engine/daemon/proxy/)

注意：互联网上流传着这样的写法，这是完全错误的，会导致Docker Desktop无法运行
```json
"proxies": {
  "default": {
    "httpProxy": "http://127.0.0.1:7890",
    "httpsProxy": "http://127.0.0.1:7890"
  }
}
```

### 运行一个容器时设置代理

```bash
docker run -r http_proxy=http://xxx.xxx.xxx.xxx:7890 https_proxy=https://xxx.xxx.xxx.xxx:7890 <image_name>
```

如果代理服务器在你的宿主机上，你可能需要查询主机ip

查询主机ip相关命令如下，三者都可
```bash
ping host.docker.internal
echo $DOCKER_HOST
grep docker /etc/hosts
```

下载`ping`工具命令
```bash
apt install iputils-ping
```

## 复制一个文件到一个容器内部

```bash
docker cp /path/to/local/file.txt my_container:/path/in/container
```

## docker容器使用宿主机的cuda

```bash
docker run --gpus=all <image_name>
```

如果你想测试一下容器能否使用宿主机的cuda，这里有一个测试命令
```bash
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```

详细内容请查看[官方文档](https://docs.docker.com/desktop/features/gpu/)

## 编写Dockerfile中的COPY时，排除某些文件

在同级目录写一个.dockerignore内容如下
```
media/
```

## 相关链接

本博客参考了[北京航空航天大学《云计算技术基础》课程实验手册](https://scs.buaa.edu.cn/doc/cloud-labs/cloud/container_docker/)

为 Docker Hub 配置国内镜像[教程](https://yeasy.gitbook.io/docker_practice/di-yi-bu-fen-ru-men-pian/03_install/3.9_mirror)
