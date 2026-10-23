---
title: Docker
tags:
  - Docker
categories:
  - Tool
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
  -p 127.0.0.1:5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:latest

# 数据库不应对本机之外开放，所以端口只绑定回环地址 127.0.0.1，只有宿主机自己能连。
# 写成 -p 5432:5432 会发布到宿主机的所有网卡上，能访问宿主机 5432 的机器都能连上；
# 而且这条转发规则由 Docker 直接写进 iptables，宿主机的 ufw、firewalld 默认不阻止，靠主机防火墙补救没有用。
# 只给同一台机器上的其他容器使用时，不写 -p，改用同一个自定义网络，按容器名连接。
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

Dockerfile 是一个纯文本文件，它按顺序写清四件事：用哪个基础镜像、装哪些依赖、把哪些文件拷进镜像、容器启动时跑什么。`docker build` 从上到下执行这些指令，产出一个镜像；`docker run` 再拿这个镜像启动一个容器。

整个流程只有三步：

1. 写 `Dockerfile`，描述应用要怎么搭起来。
2. 执行 `docker build`，得到一个镜像。镜像是一个只读模板，自己不会运行。
3. 执行 `docker run`，用镜像启动容器。容器才是真正跑起来的实例。

![[../assets/dockerfile-to-container.svg]]

## 4.1. 核心指令

指令按执行时机分成两类：`FROM`、`WORKDIR`、`COPY`、`RUN`、`ENV`、`EXPOSE` 在构建镜像时执行，结果留在镜像里；`CMD` 在容器启动时执行，不改变镜像。下面每条给出语法、行为和边界，语法里的尖括号是占位符，实际写的时候替换成具体值。

### 4.1.1. FROM：指定基础镜像

语法：`FROM <image>[:<tag>]`

指定基础镜像，之后所有指令都建立在它之上，因此必须是文件里的第一条非注释指令。`<tag>` 省略时用 `latest`，这个标签会随上游更新而移动，正式环境应写明具体版本。一个 Dockerfile 可以出现多个 `FROM`，每个开启一个独立的构建阶段。

```dockerfile
FROM python:3.12-slim
```

### 4.1.2. WORKDIR：切换工作目录

语法：`WORKDIR <dir>`

设置工作目录，目录不存在时自动创建。之后 `RUN`、`COPY`、`CMD` 里的相对路径都以它为基准，下面的 `COPY` 目标 `.` 就是 `/app`。可以出现多次，后面的相对前一条继续解析。

```dockerfile
WORKDIR /app
COPY app.py .        # 目标 . 就是 /app
```

### 4.1.3. COPY：复制文件进镜像

语法：`COPY <src>... <dest>`

从构建上下文复制文件或目录进镜像。`<src>` 相对构建上下文目录（`docker build` 末尾那个路径），不能指向它之外，`COPY ../x .` 会直接报错。`<dest>` 以 `/` 开头是绝对路径，以 `/` 结尾按目录处理，其余情况相对当前 `WORKDIR`。

```dockerfile
COPY requirements.txt .         # 复制单个文件到 WORKDIR
COPY src/ /app/src/             # 复制目录到镜像里的绝对路径
```

### 4.1.4. RUN：构建时执行命令

语法：`RUN <command>` 或 `RUN ["<executable>", "<arg>", ...]`

在构建镜像时执行命令，产生的结果（装上的软件包、生成的文件）会留在镜像里；容器启动时不会重跑。装完再清理必须写在同一条 `RUN` 里用 `&&` 串起来，分成两条的话，前一条产生的文件已经留在镜像中，删掉也减不下去体积。

```dockerfile
RUN pip install -r requirements.txt                # shell form，经过 /bin/sh
RUN ["pip", "install", "-r", "requirements.txt"]   # exec form，不经过 shell，不解析管道和变量
```

### 4.1.5. ENV：设置环境变量

语法：`ENV <key>=<value> [<key>=<value> ...]`

设置环境变量并写进镜像。构建期它后面所有 `RUN` 都能读到，容器运行期应用进程也能读到，适合放构建和运行共用的配置。它的值保存在镜像元数据里，`docker inspect` 就能看到，密钥不要写在这里。

```dockerfile
ENV APP_ENV=production LOG_LEVEL=debug
```

### 4.1.6. EXPOSE：声明监听端口

语法：`EXPOSE <port>[/<protocol>]`

声明容器运行时会监听哪个端口，只写进镜像元数据，供阅读的人和 `docker run -P`（把声明的端口映射到宿主机随机端口）参考。它本身不开放端口，也不做映射：写了 `EXPOSE 5000` 仍然要加 `-p 8000:5000` 才能从宿主机访问，不写 `EXPOSE` 加上 `-p` 一样能访问。

```dockerfile
EXPOSE 5000        # 协议省略时默认 tcp
```

### 4.1.7. CMD：设置启动命令

语法：`CMD ["<executable>", "<arg>", ...]` 或 `CMD <command> <arg>`

容器每次启动时执行的默认命令，不参与构建，同一个 Dockerfile 里只有最后一条 `CMD` 生效。exec form 不经过 shell，因此管道、`&&`、`$VAR` 都不会被解释，需要这些特性时写 shell form。运行时在镜像名后面追加命令会覆盖它，`docker run my-flask-app echo hi` 执行的会是 `echo hi`。

```dockerfile
CMD ["python", "app.py"]     # exec form，推荐
CMD python app.py            # shell form，经过 /bin/sh -c
```

## 4.2. 一个完整的例子：Python Flask 应用

目录结构：

```text
my-flask-app/
├── app.py              # 应用代码
├── requirements.txt    # 依赖清单
└── Dockerfile          # 构建配置
```

`app.py`：

```python
from flask import Flask

app = Flask(__name__)


@app.get("/")
def hello():
    return "Hello from Docker!"


if __name__ == "__main__":
    # 监听 0.0.0.0，不能写 127.0.0.1：
    # 容器里的 127.0.0.1 只指向容器自己，从宿主机映射进来的请求到不了那里
    app.run(host="0.0.0.0", port=5000)
```

`requirements.txt`：

```text
flask==3.0.3
```

`Dockerfile`：

```dockerfile
# 1. 基础镜像：官方 Python 3.12 的精简版，python 和 pip 都已经装好
FROM python:3.12-slim

# 2. 工作目录：之后每条指令都在 /app 下执行
WORKDIR /app

# 3. 先复制依赖清单并安装依赖
#    代码会频繁改动，依赖清单很少改动。把装依赖放在复制代码之前，
#    改代码时这一步能直接命中构建缓存，不必重新下载依赖
COPY requirements.txt .
RUN pip install -r requirements.txt

# 4. 再复制源代码
COPY app.py .

# 5. 声明容器监听的端口。真正的端口映射由 docker run 的 -p 决定
EXPOSE 5000

# 6. 容器启动时执行的命令
CMD ["python", "app.py"]
```

缓存逐条比对：某条指令的输入没变，Docker 就复用上一次的结果。依赖清单没改时，中间的 `RUN pip install` 会被直接跳过，这也是两个 `COPY` 要分成两次写的原因。

## 4.3. 构建镜像

在 `my-flask-app` 目录下执行：

```bash
docker build -t my-flask-app .
```

- `-t my-flask-app` 给镜像起名字，后面 `docker run` 用它引用。写成 `-t my-flask-app:1.0` 可以同时带上版本号。
- 末尾的 `.` 是构建上下文目录，也是 `COPY` 查找文件的起点，这里表示当前目录。

改完 `app.py` 要重新执行一次 `docker build`，新镜像才会包含新代码；用同一个名字再构建，新镜像会顶掉旧镜像的标签。

## 4.4. 运行容器

```bash
docker run -d -p 8000:5000 --name my-flask-server my-flask-app
```

- `-d` 后台运行，容器的输出不会占住当前终端。
- `-p 8000:5000` 端口映射，格式是 `宿主机端口:容器端口`。左边 8000 是对外访问用的端口，右边 5000 是 `app.py` 里监听的端口，两者可以不同。
- `--name my-flask-server` 给容器起名字，之后用它代替一长串容器 ID。
- 最后的 `my-flask-app` 是镜像名，和 `docker build -t` 里的名字对应。

启动后在宿主机浏览器打开 `http://127.0.0.1:8000`，会看到 `Hello from Docker!`。

容器没按预期工作时，先用两条命令看状态和输出：

```bash
docker ps                     # 容器还在不在运行
docker logs my-flask-server   # 容器打印了什么，启动报错通常在这里
```
