---
layout: post
title: Docker重点总结(7) ---- Dockerfile命令
tags:
- linux
- docker
---

# COPY
```docker
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```
和 `RUN` 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match 规则

`<目标路径>` 可以是容器内的**绝对路径**，也可以是**相对于工作目录的相对路径**（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录**不存在**会在复制文件前先行**创建缺失目录**。

此外，还需要注意一点，使用 `COPY` 指令，源文件的各种**元数据都会保留**。比**如读、写、执行权限、文件变更时间等**。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

# ADD
`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。

比如 `<源路径>` 可以是一个 `URL`,下载后的文件权限自动设置为 `600`。如果这并不是想要的权限，那么还需要增加额外的一层 `RUN` 进行权限调整，另外，如果**下载**的是个压缩包，需要解压缩，也一样还需要额外的一层 `RUN` 指令进行解压缩。所以不如全部通过`RUN`执行(wget, tar),多此一举

如果`<源路径>`是个`tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下,将会自动解压缩这个压缩文件到 `<目标路径>`。注意上一段说需要手动解压的是**下载**的压缩包

需要注意的是，`ADD` 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。
因此在 `COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`。

# CMD
`CMD` 指令的格式和 `RUN` 相似，也是两种格式：
```docker
CMD <命令>  # shell 格式
CMD ["可执行文件", "参数1", "参数2"...]  # exec 格式
CMD ["参数1", "参数2"...]  # 参数列表格式, 在指定了 ENTRYPOINT 指令后，用 CMD 指定具体的参数
```

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。`CMD` 指令就是用于指定默认的**容器主进程**的**启动命令**的。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，`ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 `bash`。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。

如果使用 `shell` 格式的话，实际的命令会被包装为 `sh -c `的参数的形式进行执行。比如:
`CMD echo $HOME` == `CMD ["sh", "-c", "echo $HOME"]`

#### 应用的前台执行
很多人可能会这样使用nginx服务
`CMD service nginx start`
实际这个操作是无效的，`Docker` 不是虚拟机，容器中的应用都应该以**前台执行**，而不是像虚拟机、物理机里面那样，用 `systemd` 去启动后台服务，**容器内没有后台服务的概念**。

对于容器而言，其**启动程序**就是容器应用进程，**容器就是为了主进程而存在的**，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

`CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 `sh`。那么当 `service nginx start` 命令结束后，`sh` 也就结束了，`sh` 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 `nginx` 可执行文件，并且要求以**前台形式运行**。
```docker
CMD ["nginx", "-g", "daemon off;"]
```

这里提示一下：
> CMD ENTRYPOINT 都是单句有效的，所以如果有多句声明只是最后一句有效

# ENTRYPOINT
ENTRYPOINT格式和RUN一样，不赘述
`ENTRYPOINT` 的目的和 `CMD` 一样，都是在**指定容器启动程序及参数**。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令，换句话说实际执行时，将变为：
```
<ENTRYPOINT> "<CMD>"
```

那么docker开放这个属性有何作用呢，这有个例子：
构建一个查询自己公网IP的镜像,起名为myip
```docker
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "https://ip.cn" ]
```
使用`docker run myip`即可查看自己的公网IP

但是，由于curl命令可以带参数，例如我们想要使用`-i`获取响应头信息，如果直接使用`docker run myip -i`显然是无效的，因为我们说过run最后的参数是执行的CMD命令，这个显然是把`-i`代替了默认的`curl -s https://ip.cn`，所以ENTRYPOINT可以很好的接入

将Dockerfile的CMD替换为ENTRYPOINT，再使用`docker run myip -i`就可以把`<CMD> -i`传给`<ENTRYPOINT> curl -s https://ip.cn`

# ENV
```docker 
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```
设置docker容器的环境变量
这不做过多说明，只是要注意这个也是一条语句构建一层镜像，所以也是尽量使用一条语句完成，如果需要空行，直接使用\
```docker
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

# ARG
和ENV一样是用来设置环境变量的，但是在未来容器运行中这些变量是不会存在的。使用build的`--build-arg <参数名>=<值>`用于覆盖ARG的参数。需要注意的是go 1.13之前，使用`--build-arg`指定的参数必须是ARG声明过的

**但是不要因此使用ARG传递密码**，因为这个在`docker history`还是能看到的。

# VOLUME
用于使用**匿名卷**
```docker
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```
路径为镜像内文件系统的绝对路径，或WORKDIR的相对路径

之前我们说过，容器运行时应该尽量保持容器存储层不发生写操作,为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，我们可以事先指定某些目录挂载为**匿名卷**,这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```docker 
VOLUME /data
```

注意，这里的 `/data` 目录就会在运行时自动挂载为**匿名卷**，任何向 `/data` 中写入的信息都**不会**记录进容器**存储层**，从而保证了容器存储层的**无状态化**。
运行这个容器后，使用`docker volume ls`可以看到一个很长的匿名volume
但是如果容器被删除，其volume内容也会被删除，所以如果想接入这个挂载点的话，需要使用`docker run -v <volume>:/data`来代替默认的匿名卷

# EXPOSE
用于**告知**使用者该镜像需要暴露哪个端口用于服务
```docker
EXPOSE <端口1> [<端口2>...]
```

这个仅仅是**告知**的作用，并不会因为配置了就能自动开放哪个端口，想使用该端口可以用`docker run -p <host_port>:<expose_port>`，当然如果使用`-P`采用随机端口映射的话，docker肯定需要一个说明，告知容器需要暴露哪个端口，所以采用这种方法则必须声明`EXPOSE`

# WORKDIR
用于指定**工作目录**，即在此目录下执行后续命令
```docker 
WORKDIR <path>
```

很多人一开始会使用
```docker 
RUN /app
RUN mkdir aaaa
```
希望在`/app`目录下建立`aaaa`文件夹，但是经过之前的学习我们知道，每一条`RUN`都会构建全新的一层，并且在新的一层执行下一条命令。但是`cd`命令仅仅在**内存上改变**，所以不会被持久化到下一层。

使用`WORKDIR`可以指定执行命令的目录,并且最后一条`WORKDIR`命令指定的目录也是启动容器进入的默认目录
```docker
# 由于刚刚说了最后一条，意味着可以有多个WORKDIR
WORKDIR /app
RUN mkdir aaaa
WORKDIR /opt
RUN mkdir bbbb
```
每个`WORKDIR`都有作用域，所以建立了`/app/aaaa`和`/opt/bbbb`。但是最好不要这样使用，这个应该用于指定最终的工程的工作路径，其余的完全可以用`RUN cd xxx && \ ...`代替

# USER
和`WORKDIR`类似，用于改变后续命令的执行者
当然在该镜像中你得有此user，另外如果非root用户需要root权限不要使用`sudo`,有一些缺陷，使用`gosu`

# HEALTHCHECK
健康检查，没用过，用到再学
https://yeasy.gitbooks.io/docker_practice/image/dockerfile/healthcheck.html

# ONBUILD
https://yeasy.gitbooks.io/docker_practice/image/dockerfile/onbuild.html

# 多阶段构建
由于种种原因，例如go程序的编译需要go环境，但是带go环境的镜像很大，我们可以先编译后再放到alpine中执行，或者其他原因，我们需要多个镜像来构建一个镜像

之前没有**多阶段构建（multistage build）**时
1. 只使用一个Dockerfile
- 镜像层次多，镜像体积较大，部署时间变长(发布也只能使用golang容器发布)
- 源代码存在泄露的风险
2. 分散到多个Dockerfile
我们要分别构建每个Dockerfile，通过编写`shell`脚本将其连接起来

现在docker支持多阶段构建,例子
```docker
FROM golang:1.9-alpine as builder
RUN apk --no-cache add git
WORKDIR /go/src/github.com/go/helloworld/
RUN go get -d -v github.com/go-sql-driver/mysql
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest as prod
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/go/helloworld/app .
CMD ["./app"]
```

前半部分是使用go环境编译go的，后半部分是将编译后的可执行文件传到alpine中执行的，他们两交流主要通过`FROM xxx as xxx`来建立连接的，其实这两者间是把app这个可执行文件传递了一下

通过`COPY --target=<image>`来指定是通过~~哪个镜像传入的~~

**注意注意**：
不要这么理解，如果`builder`指的是`golang:1.9-alpine`镜像，那么这里怎么会有app呢？
所以每个`FORM`到下一个`FROM`都是一个构建镜像的作用域，而`builder`就是指的前半个golang编译完后的镜像名

所以`as`不是**别名**的意思，而是**该阶段最终构建镜像的名称**(Dockerfile内有效)，如果上一阶段没有名称但还需要`COPY`的话,使用`COPY --from=0`表示从上一阶段的构建后镜像复制

既然是多阶段的当然可以只构建一部分
`docker build --target=builder ...`
只构建golang编译的镜像