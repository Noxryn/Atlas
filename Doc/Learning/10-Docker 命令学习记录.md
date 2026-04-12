# Docker 概述

Docker 是一个开源的应用容器引擎。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。确保开发、测试、生产环境一致，

容器是完全使用沙箱机制，相互之间不会有任何接口，更重要的是容器性能开销极低。
  
## 核心概念

- 容器（Container）：轻量化的运行实例，包含应用代码、运行时环境和依赖库。基于镜像创建，与其他容器隔离，共享主机操作系统内核（比虚拟机更高效）

- 镜像（Image）：只读模板，定义了容器的运行环境（如操作系统、软件配置等）。通过分层存储（Layer）优化空间和构建速度

- Dockerfile：文本文件，描述如何自动构建镜像（例如指定基础镜像、安装软件、复制文件等）

- 仓库（Registry）：存储和分发镜像的平台，如 Docker Hub（官方公共仓库）或私有仓库（如 Harbor）

## 运行流程

![Docker_01](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/Docker_01.png)

# Docker 命令

## 辅助命令

| 命令                 | 功能                    |
| -------------------- | ----------------------- |
| docker vesion        | 版本信息                |
| docker info          | 系统信息                |
| docker <命令> --help | 帮助手册                |
| docker logs          | 查看日志                |
| docker inspect <id>  | 获取容器/镜像的详细信息 |
| docker stats         | 资源实时使用情况        |
| docker login -u name | 登录 Docker 仓库        |
| docker logout        | 登出 Docker 仓库        |

## 镜像命令

| 命令                                                                 | 功能                                        |
| -------------------------------------------------------------------- | ------------------------------------------- |
| docker images [-a /-q]                                               | 查看本地镜像 [仓库源 标签 id 创建时间 大小] |
| -a                                                                   | 显示所有镜像                                |
| -q                                                                   | 只显示镜像 id                               |
| docker searsh <name>                                                 | 搜索镜像                                    |
| docker pull <name>                                                   | 拉取镜像                                    |
| docker push <path>                                                   | 推送镜像                                    |
| docker rmi -f <id>                                                   | 删除镜像                                    |
| docker tag <id>  名称/镜像源:标签                                    | 设置镜像标签                                |
| <dest> --volumes-from <source>                                       | 数据同步                                    |
| docker commit -m="描述" -a="作者" 容器id 目标镜像名                  | 提交修改后镜像                              |
| docker save  -o <.tar> <容器名>                                      | 保存镜像                                    |
| docker load -i <.tar>                                                | 加载 镜像                                   |
| docker build -f <Dockerfile>  -t 目标镜像名:版本号 <Dockerfile 路径> | 构建镜像                                    |

## 容器命令 

| 命令                                     | 功能                                         |
| ---------------------------------------- | -------------------------------------------- |
| docker run <name>                        | 创建并启动容器                               |
| --name="Name"                            | 容器名称                                     |
| -d                                       | 后台运行                                     |
| -it                                      | 交互式运行，进入容器                         |
| -p [ip:主机端口:容器端口                 | 主机端口:容器端口                            | 容器端口]      | 指定端口 内部端口绑定到主机端口 |
| -P                                       | 随机映射主机端口                             |
| -e                                       | 配置环境变量                                 |
| --network <网络名>                       | 加入网络                                     |
| --rm                                     | 停止即删除容器                               |
| docker ps                                | 查看运行中的容器                             |
| -a                                       | 列出所有运行过的容器                         |
| -n=?                                     | 显示最近创建的容器                           |
| -q                                       | 只显示容器编号                               |
| docker exec <>                           | 容器中执行命令                               |
| docker exec -it <容器ID> /bin/bash       | 进入容器内部                                 |
| exit                                     | 停止容器                                     |
| ctrl + P + Q                             | 退出容器但不停止                             |
| docker rm <id>                           | 删除指定容器                                 |
| docker rm -f $(docker ps -aq)            | 删除所有容器                                 |
| docker start                             | restart                                      | stop <id>      | 启动                            | 重启 | 停止 容器 |
| docker top                               | 容器中进程信息                               |
| docker cp id:path   dest_path            | 内容拷贝                                     |
| docker export <容器id> <.tar>            | 导出容器                                     |
| cat <.tar>                               | docker import - <镜像>                       | 导入容器到镜像 |
| docker volume ls                         | 列出 Docker 卷                               |
| -v 容器路径                              | 匿名挂载                                     |
| -v 卷名：容器路径                        | 具名挂载                                     |
| :ro                                      | 只读                                         |
| :rw                                      | 可读可写                                     |
| -v  主机路径::容器路径                   | 指定路径挂载                                 |
| docker network ls                        | 列出 Docker 网络                             |
| docker network create -d bridge <网络名> | 创建 Docker 网络                             |
| -d    [bridge/overlay]                   | 指定网络类型                                 |
| docker port <镜像名> <端口>              | 查看端口绑定情况                             |
| docker-compose up                        | 启动多容器应用（docker-compose.yml)          |
| docker-compose down                      | 停止并删除由 docker-compose 启动的容器、网络 |

## 具名挂载 和 匿名挂载

默认挂载在`/var/lib/docker/volumes/xxx/data`  

# Dockerfile

构建 docker 镜像的构建脚本
| 指令                                        | 功能                                       |
| ------------------------------------------- | ------------------------------------------ |
| FROM <镜像名>                               | 基础镜像                                   |
| LABEL <key>-<value>                         | 键值对添加元数据                           |
| MAINTAINER <name+email>                     | 构建者（弃）                               |
| RUN                                         | 镜像中执行命令                             |
| CMD                                         | 创建时的默认命令（可覆盖）                 |
| ENTRYOPINT  <exec>,<parm1..>                | 同 CMD（不可覆盖）                         |
| EXPOSE  <端口>                              | 监听网络端口                               |
| ENY    <key>=<value>                        | 设置环境变量                               |
| ADD                                         | 将文件、目录或远程URL复制到镜像中,自动解压 |
| COPY  [--chown=<user>:<group>] <src> <dest> | 拷贝文件到镜像                             |
| VOLUME ["路径",...]                         | 载目录                                     |
| WORKDIR <$PATH>                             | 工作目录                                   |
| USER  <用户名>[:<用户组>]                   | 指定指令用户上下文                         |
| ARG  <参数名>[=默认值]                      | 传递给构建器的变量                         |
| ONBUILD                                     | 被继承时添加触放器                         |
| STOPSIGNAL                                  | 退出容器的系统调用信号                     |
| HEALTHCHECK [选项] CMD <命令>               | 定义周期性检查                             |
| SHELL                                       | 覆盖默认 shell                             |

# Compose

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose 可以使用 docker-compose.yml 文件来配置应用程序需要的所有服务。

```
version: <v> # 指定 Compose 文件格式版本
services:
  <name>: # 服务名
    build: . # 从当前目录的 Dockerfile 构建镜像
    ports:
      - <"80:80"> # 端口映射：主机端口:容器端口
    volumes:
      - <path:path> # 卷挂载：主机路径:容器路径
    depends_on: # 依赖服务
      - <>
    environment: # 环境变量
      - <xx = xx>
    networks:
      - <net_name> # 连接到自定义网络
    healthcheck: # 健康检查
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

networks: # 定义自定义网络
 <net_name>:
    driver: {bridge}

volumes: # 定义命名卷
  <data_name>:
```
