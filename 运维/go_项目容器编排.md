# go 项目容器编排

## 前置处理

服务器使用的是 ubutu，少许指令不同，不同OS换个指令即可

先查看下当前服务器是否已经安装过：MYSQL REDIS

> systemctl status mysql.service
> 
> 
> systemctl status redis.service

> service mysql stop
> 
> 
> service redis top

## 安装docker

lsb\_release \-a

> 我这里是 ubutu 20.04

注：我是全程开的ROOT，如果非ROOT 记得：sudo

如果OS里有旧的 DOCKER ，先检查一下，再删除了：

> apt\-get remove docker docker\-engine docker.io containerd runc

安装一些前置包：

> apt\-get install ca\-certificates curl gnupg lsb\-release

这里他会连带着安装一堆的依赖库

使用下面的 curl 导入源仓库的 GPG key：

> curl \-fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt\-key add \-

> apt\-get install software\-properties\-common

将 Docker APT 软件源添加到你的系统：

> add\-apt\-repository "deb \[arch=amd64\] https://download.docker.com/linux/ubuntu $\(lsb\_release \-cs\) stable"

> add\-apt\-repository
> 
> 
> "deb \[arch=amd64\] https://mirrors.aliyun.com/docker\-ce/linux/ubuntu
> 
> 
> $\(lsb\_release \-cs\)
> 
> 
> stable"

#### 安装 docker

> apt install docker\-ce docker\-ce\-cli containerd.io

查看 docker 状态

> systemctl status docker

## 安装MYSQL 5.7

下载镜像 ：

> docker pull mysql:5.7

mkdir /data/docker/mysql57/conf

mkdir /data/docker/mysql57/data

mkdir /data/docker/mysql57/log

首次启动：

docker run \-d \-p 3306:3306 \-\-name ckMysq57 

\-\-privileged=true 

\-v /data/docker/mysql57/conf:/etc/mysql/mysql.conf.d 

\-v /data/docker/mysql57/log:/var/log/ 

\-v /data/docker/mysql57/data:/var/lib/mysql 

\-e MYSQL\_USER="ckck" \-e MYSQL\_PASSWORD="123456" 

\-e MYSQL\_ROOT\_PASSWORD="123456" 

\-e \-\-character\-set\-server=utf8 

\-e \-\-collation\-server=utf8\_general\_ci 

mysql:5.7

如果启动失败：

> docker logs XXXX

## 安装 redis5

下载镜像：

> docker pull redis:5.0.3

创建挂载目录：

> mkdir \-p /data/docker/redis

创建配置文件:

> touch /data/docker/redis/redis.conf

```
loglevel notice
requirepass 123456
```

这里不用挂载了，直接启动：

> docker run \-d \-\-name ckRedis5 \-p 6379:6379
> 
> 
> redis:5.0.3 \-\-requirepass 111111

挂载式启动

> docker run \-d \-\-name ckRedis5 \-p 6379:6379
> 
> 
> \-v /data/docker/redis/redis.conf:/etc/redis/redis.conf
> 
> 
> redis:5.0.3

## 部署项目

最好先登陆，不然 docker build 会出错:

> docker login

```
#golang 环境
FROM golang:1.18-alpine AS builder
#FROM golang:1.18 AS build

#使用 alpine ，可以减少镜像大小。但是 alpine 默认的源在国内访问不了，需要修改为国内的源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

#安装编译需要的环境gcc等
RUN apk add build-base

#设置代码的工作目录，容器启动直接进入此目录
WORKDIR /app

#设置GOLANG的环境变量
ENV GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct

#复制项目代码
COPY . .

#下载 goland 依赖包
#RUN go version;go env -w GO111MODULE=on;go env -w GOPROXY=https://goproxy.cn,direct;
RUN go mod tidy;

#编译项目代码
RUN go build -o ar120

#帮助文档
#RUN go install github.com/swaggo/swag/cmd/swag@v1.7.9;
#RUN $HOME/go/bin/swag init --parseDependency --parseInternal --parseDepth 3;
#RUN swag -v 

#开放的端口号，注：这里需要看一下项目中的配置文件，要保持一致
EXPOSE 3333 5555

CMD [ "./ar120","-e","5"]
```

开始创建镜像

> docker build \-t zgoframe:v2 .

执行镜像

> docker run \-d \-p 3333:3333 \-p 5555:5555 \-\-name myself2 zgoframe:v2

进到容器里看看

> docker exec \-it 4ea6d72da1d9 /bin/sh
