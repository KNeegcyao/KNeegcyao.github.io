---
title: Docker基础入门
top_img: 'https://haowallpaper.com/link/common/file/previewFileImg/15918089447001472'
cover: >-
  https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/main_hu47e299bfffaad059de349ef77bc2cb77_42796_1600x0_resize_box_3.png
abbrlink: 8b63f587
date: 2025-05-10 15:44:52
tags:
---

# Docker核心概念

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

1. **容器（Container）**

   - **定义**：容器是镜像的运行实例，像一个独立的小房间，里面装着一个应用和它的依赖环境。
   - **形象比喻**：
     - **类比“运行的软件”**：容器就像你电脑上打开的微信程序，而镜像是微信的安装包。
     - **类比“快餐店套餐”**：容器是一份已经配好的套餐（汉堡+薯条+可乐），随时可以开吃；镜像则是套餐的标准化配方。
   - **特点**：
     - **轻量**：共享宿主机的操作系统内核，无需重复加载。
     - **隔离**：每个容器像一间带锁的屋子，互相看不到对方的数据。

2. **镜像（Image）**

   - **定义**：镜像是容器的模板，包含应用代码、依赖和配置，像一个“只读的安装包”。
   - **形象比喻**：
     - **类比“乐高说明书”**：镜像是拼装乐高的步骤图，容器是按图拼好的成品。
     - **类比“预制菜”**：镜像是工厂预制好的食材包（切好的菜+调料），容器是加热后直接上桌的菜。
   - **特点**：
     - **分层结构**：像千层蛋糕，每一层对应Dockerfile中的一个指令（如安装依赖、复制文件）。
     - **不可变**：镜像一旦构建完成，内容不可修改，只能生成新版本

3. **Dockerfile**

   - **定义**：一个文本文件，包含构建镜像的指令（如基础镜像、依赖安装、文件复制等）。

   - **形象比喻**: Dockerfile 就像一份 **“乐高拼装说明书”**，它用文字详细描述了如何从零开始打造一个“标准化产品”（镜像)

   - **示例**：

     ```dockerfile
     FROM ubuntu:22.04         # 基础镜像
     COPY . /app               # 复制文件到容器
     RUN apt-get update && \   # 安装依赖
         apt-get install -y python3
     CMD ["python3", "/app/main.py"]  # 容器启动命令
     ```

4. **仓库（Registry）**

   - **定义**：仓库是存储和共享镜像的平台，类似于“应用商店”。
   - **形象比喻**：
     - **类比“苹果App Store”**：Docker Hub 就像应用商店，里面有Nginx、MySQL等“镜像APP”供下载。
     - **类比“图书馆”**：公共仓库（如Docker Hub）是开放书库，私有仓库（如Harbor）是内部藏书室。

---

# 常见命令

| **命令**       | **说明**                       | **文档地址**                                                 |
| :------------- | :----------------------------- | :----------------------------------------------------------- |
| docker pull    | 拉取镜像                       | [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) |
| docker push    | 推送镜像到DockerRegistry       | [docker push](https://docs.docker.com/engine/reference/commandline/push/) |
| docker images  | 查看本地镜像                   | [docker images](https://docs.docker.com/engine/reference/commandline/images/) |
| docker rmi     | 删除本地镜像                   | [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) |
| docker run     | 创建并运行容器（不能重复创建） | [docker run](https://docs.docker.com/engine/reference/commandline/run/) |
| docker stop    | 停止指定容器                   | [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) |
| docker start   | 启动指定容器                   | [docker start](https://docs.docker.com/engine/reference/commandline/start/) |
| docker restart | 重新启动容器                   | [docker restart](https://docs.docker.com/engine/reference/commandline/restart/) |
| docker rm      | 删除指定容器                   | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/rm/) |
| docker ps      | 查看容器                       | [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) |
| docker logs    | 查看容器运行日志               | [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) |
| docker exec    | 进入容器                       | [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) |
| docker save    | 保存镜像到本地压缩文件         | [docker save](https://docs.docker.com/engine/reference/commandline/save/) |
| docker load    | 加载本地压缩文件到镜像         | [docker load](https://docs.docker.com/engine/reference/commandline/load/) |
| docker inspect | 查看容器详细信息               | [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) |

关系图,来源[黑马微服务]([‍‌‌‍‍﻿﻿‬‬⁠‬‌﻿﻿‬﻿﻿﻿‍‍‬‍⁠‍﻿﻿‬‍﻿‌‬day02-Docker - 飞书云文档](https://b11et3un53m.feishu.cn/wiki/MWQIw4Zvhil0I5ktPHwcoqZdnec))

![image-20250510145510929](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250510145510929.png)

---

# 数据卷

- **定义**：数据卷是容器外部的持久化存储，避免容器删除后数据丢失。是一个虚拟目录，是**容器内目录**与**宿主机目录**之间映射的桥梁.
- **形象比喻**：
  - **类比“移动硬盘”**：容器像一台电脑，数据卷是外接硬盘，电脑坏了数据还在硬盘里。
  - **类比“云笔记”**：容器重启或删除时，数据卷像云端存储，内容不会丢失。
- **使用场景**：
  - 数据库文件（如MySQL的`/var/lib/mysql`）。
  - 配置文件（如Nginx的`/etc/nginx`）。

**为什么要使用数据卷？**

因为Docker 容器的默认存储是 **临时性** 的——容器一旦被删除，内部所有数据（如数据库文件、日志等）都会丢失。数据卷（Volume）是 Docker 中解决这一问题的核心方案，它的核心价值是 **持久化存储** 和 **数据共享**。

比如更新Mysql，需要销毁旧的容器，那数据也会一并被删除。

Nginx容器运行后，如果我要修改其中的某些配置该如何做到。

**数据卷的三大核心作用**

1. **持久化数据**

- **问题**：容器像“临时沙盒”，删除后数据清零（如 MySQL 容器被删，数据库文件丢失）。
- **解决**：数据卷将数据存储在宿主机或外部存储中，独立于容器生命周期。
- **类比**：
  容器就像一台笔记本电脑，数据卷是外接移动硬盘——即使电脑坏了，数据仍在硬盘里。

2. **跨容器共享数据**

- **问题**：多个容器需要访问同一份数据（如 Web 容器读取静态资源，另一个容器处理日志）。
- **解决**：多个容器挂载同一个数据卷，实现数据共享。
- **类比**：
  数据卷像公司共享的云盘，所有员工（容器）都能访问同一份文件。

3. **提升性能**

- **问题**：容器内文件系统（如 OverlayFS）对频繁 I/O 操作（如数据库读写）性能较差。
- **解决**：通过数据卷直接挂载宿主机目录，绕过容器文件系统，提升读写速度。
- **类比**：
  数据卷像高速公路直连仓库，绕过曲折的乡间小路（容器内部文件系统）。

![image-20250510151136176](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250510151136176.png)

在上图中：

- 我们创建了两个数据卷：`conf`、`html`
- Nginx容器内部的`conf`目录和`html`目录分别与两个数据卷关联。
- 而数据卷conf和html分别指向了宿主机的`/var/lib/docker/volumes/conf/_data`目录和`/var/lib/docker/volumes/html/_data`目录

这样以来，容器内的`conf`和`html`目录就 与宿主机的`conf`和`html`目录关联起来，我们称为**挂载**。此时，我们操作宿主机的`/var/lib/docker/volumes/html/_data`就是在操作容器内的`/usr/share/nginx/html/_data`目录。只要我们将静态资源放入宿主机对应目录，就可以被Nginx代理了。

---

**数据卷的相关命令**：

| **命令**              | **说明**             | **文档地址**                                                 |
| :-------------------- | :------------------- | :----------------------------------------------------------- |
| docker volume create  | 创建数据卷           | [docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/) |
| docker volume ls      | 查看所有数据卷       | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_ls/) |
| docker volume rm      | 删除指定数据卷       | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_prune/) |
| docker volume inspect | 查看某个数据卷的详情 | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_inspect/) |
| docker volume prune   | 清除数据卷           | [docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/) |

---

**操作样例**

**1. 基础挂载：命名卷（Named Volume）**

**场景**：数据库持久化存储（如MySQL）。

```bash
 创建命名卷
docker volume create mysql_data

 启动容器并挂载命名卷
docker run -d \
  --name mysql \
  -v mysql_data:/var/lib/mysql \  # 命名卷挂载
  -e MYSQL_ROOT_PASSWORD=123456 \
  mysql:8.0
```

**说明**：

- `mysql_data` 是 Docker 自动管理的命名卷，存储在宿主机特定路径（如 `/var/lib/docker/volumes/mysql_data`）。
- 删除容器后，数据仍保留在 `mysql_data` 卷中。

---

**2. 目录挂载：宿主机目录 → 容器目录**

**场景**：开发时挂载代码目录（实现热更新）。

```bash
docker run -d \
  --name dev-server \
  -v /home/user/project:/app \  # 宿主机目录直接挂载
  -p 3000:3000 \
  node:18
```

**说明**：

- 容器内 `/app` 目录与宿主机 `/home/user/project` 目录实时同步。
- 修改宿主机代码后，容器内立即生效（无需重建镜像）。

---

#网络

**定义**:Docker 网络是容器间通信和对外服务的桥梁，理解其机制能帮助你高效管理容器互联、暴露服务及保障安全。比如像Web服务需要访问Redis数据库。可以创建个自定义的网络，将两者都加入一个网络中，从而实现多容器通信。

⚠注意：容器本身的网络IP其实是一个虚拟的IP，其值并不固定与某一个容器绑定，如果写死的话，以后这个虚拟IP有可能会发生改变。

```bash
# 1.用基本命令，寻找Networks.bridge.IPAddress属性
docker inspect mysql
# 也可以使用format过滤结果
docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' mysql
# 得到IP地址如下：
172.17.0.2
```

**常用命令**：

| **命令**                  | **说明**                 | **文档地址**                                                 |
| :------------------------ | :----------------------- | :----------------------------------------------------------- |
| docker network create     | 创建一个网络             | [docker network create](https://docs.docker.com/engine/reference/commandline/network_create/) |
| docker network ls         | 查看所有网络             | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_ls/) |
| docker network rm         | 删除指定网络             | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_rm/) |
| docker network prune      | 清除未使用的网络         | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_prune/) |
| docker network connect    | 使指定容器连接加入某网络 | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_connect/) |
| docker network disconnect | 使指定容器连接离开某网络 | [docker network disconnect](https://docs.docker.com/engine/reference/commandline/network_disconnect/) |
| docker network inspect    | 查看网络详细信息         | [docker network inspect](https://docs.docker.com/engine/reference/commandline/network_inspect/) |

Docker 默认提供多种网络模式，适用于不同场景：

| **模式**       | **描述**                                                     | **适用场景**                           |
| :------------- | :----------------------------------------------------------- | :------------------------------------- |
| **Bridge**     | 默认模式，容器通过虚拟网桥（`docker0`）连接到宿主机网络，分配独立 IP。 | 单机多容器隔离环境（如开发测试）       |
| **Host**       | 容器直接使用宿主机网络栈，无独立 IP，共享宿主机端口。        | 高性能需求（如生产环境代理、监控工具） |
| **None**       | 禁用网络，容器完全隔离，仅能通过其他方式（如共享数据卷）通信。 | 安全敏感场景（如离线数据处理）         |
| **自定义网络** | 用户创建的网络，支持更灵活的容器互联策略（DNS 自动发现、别名访问）。 | 多容器协作（如微服务架)                |

**操作样例**：

**需求**：Web 服务需要连接 Redis 数据库。
**操作**：

```bash
#创建自定义网络：
docker network create app-net

#启动 Redis 容器并加入网络：
docker run -d \
--name redis \
--network app-net redis:alpine \

#启动 Web 容器并加入同一网络：
docker run -d \
--name web \
--network app-net -p 80:5000 my-web-app\
```

**效果**：

- Web 容器可直接通过 `redis` 主机名访问 Redis 服务（无需 IP）。
- 宿主机访问 `localhost:80` 即可访问 Web 应用。
