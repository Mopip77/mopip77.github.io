---
layout: post
title: Docker重点总结(8) ---- Compose
tags:
- linux
- docker
---

# Compose简介
`Docker Compose` 是 Docker 官方编排（Orchestration）项目之一，负责**快速**的**部署分布式应用**。

Compose 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」

我们知道使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

> Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

***
Compose 中有两个重要的概念：

**服务 (service)** ：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

**项目 (project)** ：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

Compose 的默认管理对象是**项目**，通过子命令对项目中的一组容器进行便捷地**生命周期管理**。

# 使用
#### 安装
`docker for mac`, `docker for win`自带
由于docker-compose是由python编写的，所以也可用pip安装

#### 命令
```shell
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```
默认compose配置名为`docker-compose.yaml`，也可用`-f`指定
`-p`指定项目名，默认为文件夹名
详细命令：
https://yeasy.gitbooks.io/docker_practice/compose/commands.html

# 模板文件
默认的模板文件名称为 `docker-compose.yml`，格式为 `YAML` 格式。
```docker compose
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```
`version`为compose模版版本

`services`对应之前说的服务
`webapp`为一个配置容器，名字可以随便起

注意每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中再次设置。

# 模板文件配置参数
指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 `docker-compose.yml` 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。
```docker compose
webapp:
  build: ./dir
```

你也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径。
使用 `dockerfile` 指令指定 `Dockerfile` 文件名。
使用 `arg` 指令指定构建镜像时的变量。
使用 `cache_from` 指定构建镜像的缓存。
```docker compse
webapp:
  context: ./dir
  dockerfile: Dockerfile-alternate
  args:
    buildno: 1
  cache_from:
    - alpine: latest
    - core/web_app: 3.14
```
`ports`暴露端口，和`docker run -p`格式一致
```docker compose
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```
注意：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 `60` 并且没放到引号里，可能会得到错误结果，因为 YAML 会自动解析 xx:yy 这种数字格式为 `60` 进制。为避免出现这种问题，建议数字串都采用**引号包括起来的字符串格式**。

`volumes`设置数据卷挂载
```docker compose
volumes:
  - /var/lib/mysql
  - cache/:/tmp/cache
  - ~/configs:/etc/configs/:ro
```

`secrets` 存储敏感数据，例如 `mysql` 服务密码。
```docker compose
mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret

secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
`environment`设置容器内的环境变量，可以使用数组或字典表示，可以在这配置`mysql`的密码
`command` 覆盖容器启动后默认执行的命令(即Dockerfile的CMD)。

还有各种和`docker run`参数一致的参数，详情自查
https://yeasy.gitbooks.io/docker_practice/compose/compose_file.html

附带两个重要的
`restart: always`指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为 `always` 或者 `unless-stopped`。
`read_only: true`容器的文件系统只读

## 读取变量
`Compose` 模板文件支持动态读取主机的系统环境变量和当前目录下的 `.env` 文件中的变量。
```docker compose
db:
  image: "mongo:${MONGO_VERSION}"
```
如果执行 `MONGO_VERSION=3.2 docker-compose up` 则会启动一个 `mongo:3.2` 镜像的容器,若当前目录存在 `.env` 文件，执行 `docker-compose` 命令时将从该文件中读取变量。