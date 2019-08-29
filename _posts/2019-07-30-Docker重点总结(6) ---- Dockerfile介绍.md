---
layout: post
title: Docker重点总结(6) ---- Dockerfile介绍
tags:
- linux
- docker
---

# commit
在dockerfile之前我们先说说commit

镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

假如我们开了一个nginx的docker，并更改了index.html
使用`docker diff` 命令查看存储层的变化
```shell
$ docker diff webserver
C /root
A /root/.bash_history
C /run
C /usr
C /usr/share
C /usr/share/nginx
C /usr/share/nginx/html
C /usr/share/nginx/html/index.html
...
```

Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像, 换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。 
```shell
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```
由于这个官方也不推荐使用，这里不做过多解释

#### 慎用 docker commit
例如上方修改那层存储层只修改了index.html, 但是由于命令的执行，还有很多文件被改动或添加了。这还仅仅是最简单的操作，如果是安装软件包、编译构建，那会有大量的无关内容被添加进来，如果不小心清理，将会导致镜像极为臃肿。

此外，使用 `docker commit` 意味着所有对镜像的操作都是**黑箱操作**，生成的镜像也被称为 **黑箱镜像**，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间后也无法记清具体在操作的。虽然 `docker diff` 或许可以告诉得到一些线索，但是远远不到可以确保生成一致镜像的地步。这种黑箱镜像的维护工作是非常痛苦的。

而且，回顾之前提及的镜像所使用的分层存储的概念，除当前层外，之前的每一层都是不会发生改变的，换句话说，任何修改的结果仅仅是在当前层进行标记、添加、修改，而不会改动上一层。如果使用` docker commit` 制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。这会让镜像更加臃肿。

# Dockerfile
从刚才的 `docker commit` 的学习中，我们可以了解到，镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)** ，***每一条指令构建一层***，因此每一条指令的内容，就是描述该层应当如何构建。
```docker
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

#### FROM
既然是定制镜像，那一定是以一个镜像为基础。所以`FROM`就是用来指定**基础镜像**的, 因此一个 Dockerfile 中 `FROM `是必备的指令，并且必须是第一条指令。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。
不以任何系统为基础以为着只能将可执行文件夹放入其中执行，例如go编译后的可执行文件，这样构建出来的镜像特别小。所以这也是go特别适合容器微服务开发的原因。

#### RUN
在容器中执行命令, 就像直接在命令行中输入命令
```docker
RUN <命令> [arg1] [arg2] ...
```

之前说过，Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。
所以如果RUN特别多就会构建出特别多的镜像，这是无意义切浪费的，徒增构建时间后冗余存储。
甚至这个不是你能忍受就行的，`Union FS` 是有最大层数限制的，最大127层
使用`docker image history <image>`看看你的image有几层了

所以`RUN`命令尽量用串联的写法
```docker
RUN echo 1 && \
    mkdir ff && \
    buildDeps='gcc libc6-dev make wget' && \
    apt-get install -y $buildDeps && \
    ...
    apt purge -y --auto-remove $buildDeps
```
编写dockerfile的时候需要提醒自己**不是在写shell脚本**，而是**定义每一层的构建**

还有一点，注意这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。**这是很重要的一步**，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。

#### 构建镜像
在 `Dockerfile` 文件所在目录执行：
```shell
# docker build [选项] <上下文路径/URL/->

$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
 ---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 9cdc27646c7b
 ---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```
从命令的输出结果中，我们可以清晰的看到镜像的构建过程。在 `Step 2` 中，如同我们之前所说的那样，`RUN` 指令启动了一个容器 `9cdc27646c7b`，执行了所要求的命令，并最后提交了这一层 `44aa4490ce2c`，随后删除了所用到的这个容器 `9cdc27646c7b`。

#### 构建上下文
注意我们上面build命令最后的参数是 `./` 指向当前文件夹,不少初学者以为这个路径是在指定 `Dockerfile` 所在路径,其实这是指定上下文命令

首先我们要理解 `docker build` 的工作原理。Docker 在运行时分为 `Docker 引擎`（也就是服务端**守护进程**）和**客户端**工具。Docker 的引擎提供了一组 `REST API`，被称为 `Docker Remote API`，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 `C/S` 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

所以我们在build的时候**并非在本地构建**，而是将**上下文的内容打包发给服务器端**，再由服务器端构建(即使运行在本地也是由本地的客户端发给服务端)，所以上下文也就是你此次构建的本地scope

所以很多人一开始会纳闷为什么`COPY ../a.py /usr/`， `ADD /opt/* /opt`会失败，因为这**超出了上下文的范围**，如果真的需要那些文件，应该将它们复制到上下文目录中去。

并且，如果仔细观察能发现之前的build输出第一行就是 `Sending build context to Docker daemon 2.048 kB`，现在看来就很明显了，另外充分理解上下文是很有必要的，不然有人因为`ADD /opt/* /opt`不成功而直接把Dockerfile放到根目录去构建，于是会打包整个硬盘并向服务器发送几十G甚至更多的东西

一般来说应该会将 `Dockerfile` 置于一个空目录下, 或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。并且可以用 .gitignore 一样的语法写一个 .dockerignore。

事实上，由于没有指定Dockerfile的路径，所以很多人以为那个./就是指定Dockerfile的路径，事实上这只是默认方法，即**上下文目录中的Dockerfile文件**，当然可以用`-f`参数指定dockerfile路径，并且这个不是上下文约束的`-f ../dockerfile_v1.1`

#### 其他build方法
刚刚说过了docker是c/s架构的，所以网络构建也是信手拈来，但是不常用，这里不做说明，详细请搜素

git构建，tar包构建，stdin构建，