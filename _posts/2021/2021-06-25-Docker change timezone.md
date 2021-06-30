---
layout: post
title: Docker容器修改时间（时区和本地时间）
category: docker
tags: [docker]
keywords: docker,Docker Compose,使用,介绍
excerpt: Docker容器修改时间（时区和本地时间）。

---

## Docker容器修改时间（时区和本地时间）



我看到很多教程的内容对于如何修改`Docker`下的时间有些过时，这里我总结记录下，在`CentOS 7`和`Ubuntu 18.04.2 LTS` 测试通过

# 容器运行前

在容器运行前，如果你用的是`Dockerfile`，那么你可以根据不同的基础镜像进行对应的设置。

# Dockerfile模式

对应的基础镜像不同，那么命令也会有不同，如下所示：

### Ubuntu

```shell
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' > /etc/timezone
dpkg-reconfigure --frontend noninteractive tzdata
```

### CentOS

```shell
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' > /etc/timezone
```

Dockerfile中配置时区
在Dockerfile中，可以通过如下方式添加中国时区：
```shell
FROM docker.io/centos
MAINTAINER DAVID

# 使用RUN命令设置时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai'>/etc/timezone
```


# Run模式

如果用的是run运行，那么加上如下2句代码在你的启动命令中

```shell
docker run  -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime ....
```

这个前提是你的系统对应的2个配置，是配置好了的。

- `/etc/timezon`
- `/etc/localtime`

# 容器运行后

如果你的容器已经运行了，那么只有进入容器内部进行修改，不同的底层镜像会有不同的命令，所以请自行选择，这里我拿`CentOS`举例

```shell
docker exec -it docker-name /bin/bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' > /etc/timezone
```

# docker-compose.yaml中配置时区
但如果docker程序是以docker-compose.yaml构建，那么时区这块需要如何设置呢？

经过实测，如下两种方式都可行。
```shell
方式一：
environment:
  - SET_CONTAINER_TIMEZONE=true
  - CONTAINER_TIMEZONE=Asia/Shanghai
	  
方式二：
environment:
  - TZ=Asia/Shanghai
```

不积跬步，无以至千里。不积小流，无以成江海。