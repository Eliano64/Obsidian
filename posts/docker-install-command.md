---
title: docker install command
tags:
  - Docker
  - Tool
categories:
  - tool
date: 2025-09-26 00:00:00
katex: true
---

# 1. 国内ubuntu服务器安装docker(使用aliyun镜像源,root用户)

## 1.1. 安装

```bash
apt update
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add - # 可能需要先安装curl
add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" # 可能需要先apt install software-properties-common
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 1.2. 换源

```bash
vim /etc/docker/daemon.json #可能需要先安装vim
```

然后添加国内的镜像仓库

```json
{
	"registry-mirrors": [
	    "https://docker.registry.cyou",
	    "https://docker-cf.registry.cyou",
	    "https://dockercf.jsdelivr.fyi",
	    "https://docker.jsdelivr.fyi",
	    "https://dockertest.jsdelivr.fyi",
	    "https://mirror2.aliyuncs.com",
	    "https://dockerproxy.com",
	    "https://mirror.baidubce.com",
	    "https://docker.m.daocloud.io",
	    "https://docker.nju.edu.cn",
	    "https://docker.mirrors.sjtug.sjtu.edu.cn",
	    "https://docker.mirrors.ustc.edu.cn",
	    "https://mirror.iscas.ac.cn",
	    "https://docker.rainbond.cc"
	]
}
```

## 1.3. 启动

```bash
systemctl start docker
systemctl enable docker # 开机自启
```

# 2. 部分常用命令

docker的核心概念围绕着镜像image、容器container、卷volume。这里不对这些概念进行说明，仅展示命令。

## 2.1 image

### 2.1.1. 查看本地已有镜像

```bash
docker images
```

或者更推荐：

```bash
docker image ls
```

会显示 REPOSITORY、TAG、IMAGE ID、大小等信息。

### 2.1.2. 拉取镜像

以postgres为例

```bash
docker pull postgres:latest
```

也可以指定版本：

```bash
docker pull postgres:15
```

### 2.1.3. 删除镜像

删除某个镜像：

```bash
docker rmi <IMAGE_ID>
```

如果镜像正在被容器使用，要么先停止并删除容器，要么加 `-f` 强制删除：

```bash
docker rmi -f <IMAGE_ID>
```

### 2.1.4. 清理不用的镜像

清理所有悬空镜像（dangling，名字是 `<none>` 的）：

```bash
docker image prune
```

清理所有不用的镜像（慎用，会清理未使用的容器相关镜像）：

```bash
docker image prune -a
```

### 2.1.5. 镜像构建

某目录下有 `Dockerfile`可以自己构建镜像：

```bash
docker build -t myapp:1.0 <dir>
```

这里 `-t` 给镜像加名字和版本号。

### 2.1.6. 镜像保存与导入

导出镜像到文件：

```bash
docker save -o myapp.tar myapp:1.0
```

从文件导入镜像：

```bash
docker load -i myapp.tar
```

## 2.2 container

容器就相当于一个配置好的虚拟机

### 2.2.1. 查看容器

- 查看正在运行的容器：

```bash
docker ps
```

- 查看所有容器（包括已停止的）：
    

```bash
docker ps -a
```

### 2.2.2. 启动/停止/重启容器

- 启动已存在的容器：
    

```bash
docker start <CONTAINER_ID 或 NAME>
```

- 停止容器：
    

```bash
docker stop <CONTAINER_ID 或 NAME>
```

- 重启容器：
    

```bash
docker restart <CONTAINER_ID 或 NAME>
```

### 2.2.3. 进入容器

- 交互式进入容器（常用）：
    

```bash
docker exec -it <CONTAINER_ID 或 NAME> bash
```

- 也可以使用别的指令，如以下是连接容器里的PostgreSQL：

```bash
docker exec -it <id/name> psql -U postgres -d postgres
```

### 2.2.4. 查看容器日志

```bash
docker logs <CONTAINER_ID 或 NAME>
```

实时查看日志：

```bash
docker logs -f <CONTAINER_ID 或 NAME>
```

### 2.2.5. 删除容器

- 删除已停止的容器：
    

```bash
docker rm <CONTAINER_ID 或 NAME>
```

- 删除所有已停止的容器：
    

```bash
docker container prune
```

### 2.2.6. 容器与镜像关系

- 基于镜像创建容器：
    

```bash
docker run -d --name mycontainer myimage:latest
```

仅是一个最小化的指令，实际可能还需要`-p`指定端口映射，`-e`指定容器运行的一些环境变量，`-v`把容器的数据（虚拟机的一个目录）映射到宿主机的卷里，比如：

```bash
docker run -d \
  --name pgdb \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret123 \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:latest
```

- 查看容器使用的镜像：
    

```bash
docker inspect <CONTAINER_ID> | grep Image
```

### 2.2.7. 其他常用

- 查看容器的资源使用情况：
    

```bash
docker stats
```

- 拷贝文件：  
    从宿主机拷贝到容器：
    

```bash
docker cp localfile <CONTAINER_ID>:/path/in/container
```

从容器拷贝到宿主机：

```bash
docker cp <CONTAINER_ID>:/path/in/container localfile
```

## 2.3 volume

### 2.3.1. 查看卷

- 查看所有卷：
    

```bash
docker volume ls
```

- 查看某个卷的详细信息（挂载点、创建时间等）：
    

```bash
docker volume inspect <VOLUME_NAME>
```

### 2.3.2. 创建卷

```bash
docker volume create myvolume
```

也可以带选项：

```bash
docker volume create --driver local --opt type=none --opt device=/path/on/host --opt o=bind myvolume
```

（把宿主机目录绑定为卷）

### 2.3.3. 删除卷

- 删除某个卷：
    

```bash
docker volume rm <VOLUME_NAME>
```

- 删除所有未使用的卷：
    

```bash
docker volume prune
```

> 注意：正在使用的卷不能删除。

### 2.3.4. 在容器中挂载卷

运行容器时挂载：

```bash
docker run -d \
  --name pgdb \
  -e POSTGRES_PASSWORD=secret123 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:latest
```

这里 `pgdata` 就是一个卷，数据会保存在宿主机 `/var/lib/docker/volumes/pgdata/_data`。

### 2.3.5. 卷与文件拷贝

- 宿主机直接访问卷的数据：
    
```bash
ls /var/lib/docker/volumes/<VOLUME_NAME>/_data
```

- 也可以用容器操作卷的数据，例如：
    

```bash
docker run --rm -v <VOLUME_NAME>:/data busybox ls /data
```

### 2.3.6. 查看卷占用空间

Docker 本身不能直接显示卷的大小，可以通过宿主机命令：

```bash
du -sh /var/lib/docker/volumes/*
```
## 2.4. docker system

`docker system` 是 Docker 提供的一个顶级管理命令，用来查看和清理 Docker 的整体资源（镜像、容器、卷、构建缓存等）。

### 2.4.1. 查看 Docker 整体资源占用

```bash
docker system df
```

类似 `df -h`，会显示：

* 镜像大小
* 容器占用空间
* 卷占用空间
* 构建缓存大小

### 2.4.2. 清理未使用资源

```bash
docker system prune
```

会删除：

* 已停止的容器
* 未使用的网络
* 悬空的镜像（没有 tag 的 `<none>` 镜像）
* 构建缓存

更彻底：

```bash
docker system prune -a
```

还会删除 **所有未被容器使用的镜像**（不仅仅是 `<none>`）。

再加上卷（慎用！会清空未使用的卷里的数据）：

```bash
docker system prune --volumes
```

### 2.4.3. 查看 Docker 信息

```bash
docker system info
```

显示 Docker 运行环境的信息（版本、存储驱动、网络、CPU/内存限制等）。

# 3. docker日志管理

## 3.1. 日志文件位置

Docker 容器日志默认在：

```txt
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

每个容器都有一个 `.log` 文件，可能会无限长。

## 3.2. 方法1: 直接清空某个容器日志

找到容器 ID：

```bash
docker ps -a
```

然后清空日志文件：

```bash
truncate -s 0 /var/lib/docker/containers/<container-id>/<container-id>-json.log
```

> `truncate -s 0` 会把文件清零，但不会删除文件。容器还在跑，不受影响。

## 3.3. 方法2: 批量清空所有容器日志

一行命令清理全部：

```bash
for log in $(find /var/lib/docker/containers/ -name *-json.log); do
  truncate -s 0 $log
done
```


## 3.4. 方法3: 配置日志大小限制

在运行容器时指定日志参数：

```bash
docker run -d \
  --name myapp \
  --log-opt max-size=100m \
  --log-opt max-file=3 \
  myimage
```

这样每个日志文件最大 100MB，最多保留 3 个。

使用`docker-compose.yml`可以这样写：

```yaml
logging:
  driver: "json-file"
  options:
    max-size: "100m"
    max-file: "3"
```

---

## 3.5. 方法4: 修改 Docker 全局配置（永久生效）

```bash
vim /etc/docker/daemon.json
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

然后重启 Docker：

```bash
systemctl restart docker
```

# 4. 使用 Dockerfile

Dockerfile 是一个用于构建 Docker 镜像的文本配置文件，其中包含了构建镜像所需的一系列指令。

构建镜像的核心流程通常包含三个阶段：
1.  **指定基础镜像**：选择操作系统或运行时环境作为构建起点。
2.  **构建应用环境**：安装依赖包、复制源代码、配置系统参数。
3.  **定义入口程序**：指定容器启动时默认执行的进程。

## 4.1. 核心指令详解

| 指令 | 含义 | 解释 | 示例 |
| :--- | :--- | :--- | :--- |
| **FROM** | 基础镜像 | 指定构建所使用的基础镜像，必须是 Dockerfile 的第一条非注释指令。 | `FROM python:3.9-slim` |
| **WORKDIR** | 工作目录 | 设置镜像内的当前工作目录，后续指令（RUN, CMD, COPY 等）将在此目录下执行。 | `WORKDIR /app` |
| **COPY** | 复制文件 | 将宿主机构建上下文中的文件或目录复制到镜像文件系统中。 | `COPY . .` |
| **RUN** | 执行命令 | 在镜像构建过程中执行 Shell 命令，常用于安装软件包或配置环境。 | `RUN pip install -r requirements.txt` |
| **CMD** | 启动命令 | 指定容器启动时默认执行的命令及参数。 | `CMD ["python", "app.py"]` |
| **EXPOSE** | 暴露端口 | 声明容器运行时监听的网络端口（主要用于文档说明和映射建议）。 | `EXPOSE 8080` |
| **ENV** | 环境变量 | 设置持久化的环境变量，可在构建期及运行期使用。 | `ENV APP_ENV=production` |

## 4.2. 实战：编写 Go (Gin 框架) 应用的 Dockerfile

Go 语言是编译型语言，最佳实践是使用 **多阶段构建 (Multi-stage Build)**。这样可以在第一阶段编译代码，在第二阶段仅打包二进制文件，从而获得极小的镜像体积（通常仅 10MB+）。

假设目录结构如下：
```text
my-app/
├── main.go              # 应用源代码
├── go.mod              # 依赖定义
├── go.sum              # 依赖校验
└── Dockerfile          # 构建配置文件
```

### 4.2.1. 编写 Dockerfile

打开 `Dockerfile`，写入以下内容：

```dockerfile
# ------------- 第一阶段：构建阶段 (Builder) -------------
# 使用官方 Go 镜像作为构建环境
FROM golang:1.23-alpine

# 设置工作目录
WORKDIR /app

# 1. 预下载依赖 (利用 Docker 缓存机制的核心步骤)
# 先仅复制依赖定义文件 (go.mod 和 go.sum)
# 为什么不直接 COPY . . ?
# 因为 Docker 构建是分层的。如果先 COPY 所有文件，只要源代码改了一个标点符号，
# 这一层缓存就会失效，导致下一行 RUN go mod download 必须重新执行（重新下载所有依赖）。
# 而只复制 go.mod，只要依赖没变，Docker 就会复用之前的下载缓存，瞬间完成构建。
COPY go.mod go.sum ./

# 下载依赖
RUN go mod download

# 编译应用
# 依赖准备好后，再复制源代码
COPY . .
# 编译为静态链接的二进制文件，命名为 server
RUN go build -o server .

# 声明端口
EXPOSE 8080

# 启动应用
CMD ["./server"]
```

## 4.3. 构建与运行

### 4.3.1. 构建镜像

```bash
# 构建名为 my-go-app 的镜像
docker build -t my-go-app .
```

### 4.3.2. 运行容器

```bash
# 启动容器，映射宿主机 8080 端口到容器 8080 端口
docker run -d -p 8080:8080 --name my-go-server my-go-app
```