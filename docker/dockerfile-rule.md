# 如何制作最合适docker镜像

## 控制镜像大小
* 优先选取基于alpine的镜像
  查看tomcat镜像：https://hub.docker.com/_/tomcat?tab=tags （docker hup 中搜索tomcat镜像）
  如tomcat8.5的镜像
```
8.5-jre11 204 MB
Last update: 13 days ago
```
而基于alpine的tomcat镜像
```
8.5-jre8-alpine 72 MB
Last update: 10 days ago
```
一眼就能看出来基于alpine的镜像比默认镜像少了130多M
但alpine镜像里面缺少了很多东西，设置不支持bash命令，我们可以在制作镜像时，一一给安装上，安装完成后删除缓存文件，就是安装文件；

## 选取国内的镜像源

如清华大学、阿里云等

*  Dockerfile示例：

```
FROM alpine:3.7

MAINTAINER chenzhenglin
#更新Alpine的软件源为国内（清华大学）的站点，因为从默认官源拉取实在太慢了。。。
RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.7/main/" > /etc/apk/repositories
# 注意alpine的版本号 可能有差异，链接地址源最好用浏览器打开查看验证一下； 或用latest代替
RUN apk update \
        && apk upgrade \
        && apk add --no-cache bash \
        bash-doc \
        bash-completion \
        && apk add net-tools vim \
        && rm -rf /var/cache/apk/* \
        ……
```
apk add --no-cache不使用本地缓存安装包数据库，直接从远程获取安装包信息安装。这样我们就不必通过apk update获取安装包数据库了，&&合并命令 好处是，多条命令合并成一条命令，减少docker images的层

如果基于centos或Ubuntu的系统   最好也要把软件源换成国内的

```bash
FROM centos:7.6.1810
RUN mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.bak && \
    curl -s  http://mirrors.aliyun.com/repo/Centos-7.repo  -o  /etc/yum.repos.d/CentOS-Base.repo && \
    curl -s  http://mirrors.aliyun.com/repo/epel-7.repo -o  /etc/yum.repos.d/epel-7.repo && \
    yum repolist && \
```

* 注意：Alpine 镜像中的 telnet 在 3.7 版本后被转移至 busybox-extras 包中，所以apk add telnet 会报错

### 如下，一个基于tomcat:8.5-alpine镜像的Dockerfile，并验证通过
```
FROM tomcat:8.5-alpine

MAINTAINER chenzhenglin
#更新Alpine的软件源为国内（清华大学）的站点，因为从默认官源拉取实在太慢了…… alpine版本要从dockerfile里面查找到
RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/latest-stable/main/" > /etc/apk/repositories
# 注意alpine的版本号 可能有差异
RUN apk update \
    && apk upgrade \
    && apk add --update --no-cache net-tools vim busybox-extras curl \
    && rm -rf /var/cache/apk/*
```
## 查看镜像的Dockerfile
* 在docker  hub 里面找到所需镜像后，在description的tab页，选择需要的版本，点击，一般会自动跳转到其githup源码界面；
  如：https://hub.docker.com/_/tomcat?tab=description
  如下Dockerfile文件
```bash
FROM openjdk:8-jre
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
RUN mkdir -p "$CATALINA_HOME"
WORKDIR $CATALINA_HOME
……此处也省略不必要代码
RUN apt-get update && apt-get install -y --no-install-recommends \
		libapr1 \
……此处也省略不必要代码
	apt-get install -y --no-install-recommends wget ca-certificates; \
……此处也省略不必要代码
# sh removes env vars it doesn't support (ones with periods)
# https://github.com/docker-library/tomcat/issues/77
	find ./bin/ -name '*.sh' -exec sed -ri 's|^#!/bin/sh$|#!/usr/bin/env bash|' '{}' +; \
	\
……此处也省略不必要代码
CMD ["catalina.sh", "run"]
```

从这个Dockerfile文件可以看出，已经安装了wget等工具，并把默认的sh换成了bash；

如何判断tomcat里面采用哪一版本的alpine呢,激活镜像后，查看其系统版本信息
`cat /etc/os-release `
同样其他linux也可以采用此方法查看版本号：
`cat /etc/redhat|centos-release `

alpine系统添加软件的常用方法，参见链接：
https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management

### 其他精简镜像的方法
   https://mp.weixin.qq.com/s/LOXNMYtZbnYeDR2lBI56fw

