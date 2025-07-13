---
title: "Docker知识总览"
date: 2021-11-05T22:00:38+08:00
draft: false
image: 1.jpg
categories:
    - computer
---

## 概念知识
### Docker镜像
Docker镜像是由文件和元数据组成的。
  - 文件：语言环境、库、执行文件
    - 由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。
  - 元数据：环境变量、端口映射、卷等
  - 镜像信息：
    - RESPOSITORY：仓库名
    - TAG：标签
    - IMAGE ID：镜像ID
    - CREATED：创建时间
    - SIZE：所占用的空间
  - 虚悬镜像(dangling image) 
    - RESPOSITROY及TAG都为<none>
    - 使用docker image prune 可以删除
### 容器
容器是从镜像中创建，继承了他们的文件系统，并使用他们的元数据来确定其启动配置。
  - 启动时，运行一个进程，不过可派生其他进程。
  - 文件的变更通过写时复制存储在容器中，基础镜像不会受影响。
### 数据卷
数据卷 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：
 - 数据卷 可以在容器之间共享和重用
 - 对 数据卷 的修改会立马生效
 - 对 数据卷 的更新，不会影响镜像
 - 数据卷 默认会一直存在，即使容器被删除
 - 注意：数据卷 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）。
 - 数据卷 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 数据卷
### Docker网络
当 Docker 启动时，会自动在主机上创建一个 docker0 虚拟网桥，实际上是 Linux 的一个 bridge，可以理解为一个软件交换机。它会在挂载到它的网口之间进行转发。

同时，Docker 随机分配一个本地未占用的私有网段（在 RFC1918 中定义）中的一个地址给 docker0 接口。比如典型的 172.17.42.1，掩码为 255.255.0.0。此后启动的容器内的网口也会自动分配一个同一网段（172.17.0.0/16）的地址。

当创建一个 Docker 容器的时候，同时会创建了一对 veth pair 接口（当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包）。这对接口一端在容器内，即 eth0；另一端在本地并被挂载到 docker0 网桥，名称以 veth 开头（例如 vethAQI2QT）。通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。Docker 就创建了在主机和所有容器之间一个虚拟共享网络。
![image](https://gblobscdn.gitbook.com/assets%2F-M5xTVjmK7ax94c8ZQcm%2F-M5xT_hHX2g5ldlyp9nm%2F-M5xTloJ8V-9G0aacJWQ%2Fnetwork.png?alt=media)
  
### 其他知识
- Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 systemd 去启动后台服务，容器内没有后台服务的概念。

## 使用镜像的命令工具

命令 | 功能
---|---
docker pull [Registry地址[:端口]/] 仓库名[:标签] | Registry 地址拉取镜像
Docker run 仓库名:标签 [命令] | 以容器形式运行一个Docker镜像
Docker run -it 仓库名:标签 [命令] | -it表示进入交互终端
Docker run --rm 仓库名:标签 [命令] | --rm退出后删除，否则docker ps -a看得到
docker image ls | 列出已经下载下来的镜像
docker images | 同上
docker image ls [仓库名][:标签]| 列出指定仓库名或附带标签的镜像
docker image ls -q | 仅列出表格的IMAGE ID列表
docker image ls -f label=com.example.version=0.1 | -f 增加过滤器
docker image ls --format "{{.ID}}: {{.Repository}}" | 利用Go的模板语法列出信息
docker image ls --digests | 列出镜像，增加DIGEST摘要显示（确保唯一）
docker image rm [选项] <镜像1> [<镜像2> ...] | 删除镜像，镜像可以是ID、仓库名:标签，摘要
docker image rm $(docker images -q redis) | 删除所有redis名称的镜像
docker image rm $(docker images -q -f before=镜像) | 删除某个镜像前面的镜像
docker run --name webserver -d -p 80:80 nginx | 指定容器名称为webserver，且配置端口
docker exec -it webserver bash | 进入容器，打开bash控制台
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]] | 保存镜像
Docker tag | 给一个Docker镜像打标签
docker system df | 便捷的查看镜像、容器、数据卷所占用的空间
docker image prune | 删除虚悬镜像(dangling image) 
docker build [选项] <上下文路径/URL/-> | 构建镜像

```
$ docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
```


### 使用Dockerfile定制镜像
指令 | 功能
---|---
FROM <基础镜像> | 基础镜像
FROM scratch | 不以任何镜像为基础，Go语言开发的应用很多会使用这种方式
RUN shell格式命令 | shell命令追加层
RUN ["可执行文件", "参数1", "参数2"] | exec格式追加层
COPY ./package.json /app/ | 其中.表示上下文目录
docker build -t nginx:v3 . | 其中.就是传入的上下文目录，一般及时Dockerfile文件所在目录
COPY [--chown=<user>:<group>] <源路径>... <目标路径> | 复制上下文目录中的文件/目录到镜像中
WORKDIR <绝对路径> | 指定某个绝对路径作为后续的相对路径
ADD [--chown=<user>:<group>] <源路径>... <目标路径> | 仅源自动解压缩时使用
CMD <shell命令> | 容器主进程的启动命令，例如ubuntu镜像默认的CMD是/bin/bash
CMD ["可执行文件", "参数1", "参数2"...] | exec 格式容器主进程的启动命令，推荐使用
ENTRYPOINT ["可执行文件", "参数1", "参数2"...] | 入口点，镜像变成命令一样使用，或者运行前的准备工作
ENV <key> <value> | ENV 设置环境变量
ENV <key1>=<value1> <key2>=<value2>... | ENV 设置环境变量
ARG <参数名>[=<默认值>] | docker build中用--build-arg <参数名>=<值>来覆盖，生效范围为下一个指令
VOLUME ["<路径1>", "<路径2>"...] | VOLUME 定义匿名卷，这样运行时，不会向容器存储层写入数据，运行命令-v可以覆盖位置
VOLUME <路径> | VOLUME 定义匿名卷
EXPOSE <端口1> [<端口2>...] | 暴露端口，容器运行时提供服务的端口，这只是一个声明
WORKDIR <工作目录路径> | 指定工作目录（或者称为当前目录），如该目录不存在，WORKDIR 会帮你建立目录
USER <用户名>[:<用户组>] | 指定当前用户，这个用户必须是事先建立好的
HEALTHCHECK [选项] CMD <命令> | 设置检查容器健康状况的命令
HEALTHCHECK NONE | 如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令\
ONBUILD <其它指令> | 后面指令，以当前镜像为基础镜像构建下一级镜像时被执行。
LABEL <key>=<value> <key>=<value> ... | 镜像以键值对的形式添加一些元数据（metadata）
SHELL ["executable", "parameters"] | 指定RUN ENTRYPOINT CMD 指令的 shell，Linux 中默认为 ["/bin/sh", "-c"]

示例
```
FROM FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

```
docker build -t nginx:v3 .
```

```
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```
```
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

```
COPY package.json /usr/src/app/
```
```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```
```
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

```
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```

```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

```
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
# 后面的指令中，可以通过$VERSION、$NAME来引用。列指令可以支持环境变量展开：
# ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN。
```

```
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

```
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

```
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

## Dockerfile多阶段构建
在Docker 17.05之后，开始支持多阶段构建 (multistage builds)。解决了之前编写2个Dockerfile的方案，一个Dockerfile把项目及其依赖库编译测试打包，一个Dockerfile构建运行镜像包，把之前的构建结果拷贝到运行环境。

我们可以使用 as 来为某一阶段命名，例如
```
  FROM golang:alpine as builder
```
一个为go程序，多阶段构建的例子：
```
FROM golang:alpine as builder

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld/

RUN go get -d -v github.com/go-sql-driver/mysql

COPY app.go .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest as prod

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=0 /go/src/github.com/go/helloworld/app .

CMD ["./app"]
```

例如当我们只想构建 builder 阶段的镜像时，增加 --target=builder 参数即可
```
  docker build --target builder -t username/imagename:tag .
```

构建时从其他镜像复制文件
```
  COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

## 构建多种系统架构支持的 Docker 镜像
参见：https://yeasy.gitbook.io/docker_practice/image/manifest

## Docker 镜像的导入和导出 docker save 和 docker load

命令示例 | 说明
---|---
docker save alpine -o filename | 把alphine镜像保存到filename文件
docker save alpine | gzip > alpine-latest.tar.gz | 使用gzip压缩，把alphine镜像保存到压缩文件中
docker load -i alpine-latest.tar.gz | 把alpine-latest.tar.gz镜像压缩文件导入到Docker系统中


## 操作镜像
命令示例 | 说明
---|---
docker run ubuntu:18.04 /bin/echo 'Hello world' | 通过镜像，输出一个 “Hello World”，之后终止容器
docker run -t -i ubuntu:18.04 /bin/bash | 启动一个 bash 终端，允许用户进行交互
docker container start 历史容器 | 将一个已经终止（exited）的容器启动运行
docker run -d ubuntu:18.04 /bin/sh -c "...脚本" | -d容器输出到容器中，会返回一个唯一的id
docker container ls | 可以看到容器id，运行中的
docker container ls -a | 查看所有容器，加了-a包括已经终止的容器
docker container logs [container ID or NAMES] | 可以获取容器的输出信息
docker container stop [container ID or NAMES] | 终止一个运行中的容器
docker exec -it 69d1 bash | 进入container ID以69d1开头的容器，打开bash，退出时，容器继续运行
docker attach -it 69d1 bash | 进入container ID以69d1开头的容器，打开bash，退出时，容器停止
docker export 7691a814370e > ubuntu.tar | 导出容器快照到本地文件
cat ubuntu.tar | docker import - test/ubuntu:v1.0 | 从容器快照文件中再导入为镜像
docker import http://example.com/exampleimage.tgz example/imagerepo | 通过指定 URL 或者某个目录来导入
docker container rm trusting_newton | 删除一个处于终止状态的NAMES为trusting_newton的容器
docker container prune | 清理所有处于终止状态的容器

## 数据管理

命令示例 | 说明
---|---
docker volume create my-vol | 创建一个名称为my-vol的数据卷，宿主机默认目录/var/lib/docker/volumes/my-vol/_data
docker volume ls | 查看所有的 数据卷
docker volume inspect my-vol | 在主机里查看指定 数据卷 的信息
docker run -v my-vol:/usr/share/nginx/html [其他省略] | 运行时加载my-vol数据卷 到容器的 /usr/share/nginx/html 目录
docker run --mount source=my-vol,target=/usr/share/nginx/html [其他省略] | 同上，另一种写法
docker inspect web | 查看web容器时，可以在Mouts中查看数据卷的具体信息
docker volume rm my-vol | 删除数据卷 
docker rm -v <容器ID或名称> | 删除容器时，同时移除数据卷
docker volume prune | 清理无主的数据卷
docker -v /src/webapp:/usr/share/nginx/html [其他省略] | 挂载一个本地主机的目录到容器中去
docker --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html [其他省略] | 同上，另一种严格写法
-v /src/webapp:/usr/share/nginx/html:ro [其他省略] | ro表示只读，不加表示读写
--mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly [其他省略] | 表示只读
-v $HOME/.bash_history:/root/.bash_history  [其他省略] | 挂载一个本地主机文件作为数据卷
--mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history [其他省略]  | 同上

## 使用网络

命令 | 功能
---|---
docker run -p [其他省略] | -p随机映射端口到内部容器开放的网络端口
docker container ls | 列表中PORTS（如0.0.0.0:32768->80/tcp）前一个表示宿主机端口
docker logs <容器ID> | 上面已描述，查看容器内的打印
docker run -p ip:hostPort:containerPort [其他省略] | 指定映射端口，-p可以多次使用
docker run -p ip::containerPort [其他省略] |  指定映射端口
docker run -p hostPort:containerPort [其他省略] |  指定映射端口
docker port <容器ID> <容器端口> | 查看当前映射的端口配置，和绑定的地址
docker network create -d bridge my-net | 创建Docker网络，-d指定类型，有bridge\overlay，my-net是名称
mount | 在容器中使用mount可查看挂载信息
docker run [其他省略] cat /etc/resolv.conf | 启动时，查看DNS配置
docker run [其他省略] -h HOSTNAME | 设定容器的主机名，写到容器内的/etc/hostname 和/etc/hosts，外包看不到
docker run [其他省略] --dns=IP_ADDRESS | 添加 DNS 服务器到容器的/etc/resolv.conf 中，等在宿主/etc/hosts 临时增加


## Dockerfile最佳实践指南
- 不常变动的部分写在dockerfile上面，以便后续变更时可以利用缓存，减少build时间。
- 编写 .dockerignore 编译镜像时，docker 先要准备编译用的context，默认情况下会把 Dockerfile所在的所在文件夹下所有文件包含进去，如果不想把src目录包含进去来加快编译速度，可以添加.dockerignore，内容如下 src/
- RUN rm xxx 删除前面层的文件不会减小镜像大小，因为包含文件的那层会一直存在，RUN set合并为一层
- 尽量不在dockerfile去修改文件权限，修改权限后的文件会生成一份新的文件导致镜像变大，修改权限最好本地直接改好或写在启动脚本中(不推荐)。
- 只复制需要的，如果可能，避免复制。在将文件复制到镜像中时，请确保对要复制的内容非常明确，避免 COPY . /home/admin/broker 这样的操作，使用COPY app/xxx.jar /home/admin/broker
- 添加文件夹到指定目录，最好填写绝对路径，如果想把kafka文件夹添加到home，要写成ADD kafka/ /home/kafka/
- 大的rpm包，如果要安装到镜像中，可以做成yum 源，yum install xx之后yum clean all，如果ADD XX.rpm /xx，这样rpm会使镜像增大。
- 大的压缩包最好是做成可以下载的文件，下载解压后删除源文件
- 多个RUN或者ENV最好合并为一个，这样生成的文件都在同一层，便于优化大小
- 去掉不必要的组件，比如 yum install vim 不安装运行不必须得vim，可以减小镜像体积
- 每次RUN的后面阶段删除不必要的文件，比如 yum install xx 安装软件后要执行 yum clean all 清除缓存，减小镜像。
- 基础镜像要指定明确的版本 FROM openjdk:latest 建议使用 FROM openjdk:8-jre-alpine
- 基础镜像如果可能，尽量使用官方发布的版本，第三方的版本有被植入病毒的风险，而且没法保证及时补丁升级和更新
- 基础镜像尽量选择小体积的发行版本，比如基于alpine linux制作的镜像，
- 分阶段编译，docker 17.05 版本以后开始支持分阶段编译，可以使用编译镜像去编译，生成的目标文件导入到运行环境镜像中，这样编译出的镜像就可以不需要一些编译工具，编译依赖，去掉编译中产生的无用文件
- WORKDIR 为 RUN CMD ENTRYPOINT指定一个默认的工作目录
- 启动脚本中最好使用 exec去运行程序
- 使用LABEL去添加一些附属信息，比如作者，联系方式，内部软件版本之类
- 使用HEALTHCHECK添加健康检查，判断拉起的容器业务是否正常

```
FROM golang:1.7.3 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]  
```

## Docker 实战与练习
[Docker 实战与练习](./Docker.pdf)
