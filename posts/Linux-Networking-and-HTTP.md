---
title: Linux Networking and HTTP
tags:
  - Network
categories:
  - Linux
date: 2026-06-26 00:00:00
katex: true
---

# Linux Networking and HTTP Notes

## Quick Lookup

| 场景 | 命令 |
|---|---|
| 查看监听端口 | `ss -tulnp` / `netstat -tulnp` |
| 查看 TCP 连接 | `ss -ant` / `netstat -ant` |
| 查端口占用 | `ss -ltnp | grep 3000` |
| 查看路由 | `ip route` / `netstat -rn` |
| 查看网卡地址 | `ip addr` |
| 测连通性 | `ping host` |
| 测 DNS | `dig domain` / `nslookup domain` |
| 调 HTTP 接口 | `curl -i URL` |
| 下载文件 | `curl -L -O URL` / `wget URL` |
| 跟踪网络系统调用 | `strace -e trace=network command` |

## 1. Network Model in Daily Debugging

排查网络问题时，可以按层次缩小范围：

| 层次 | 常见问题 | 常用命令 |
|---|---|---|
| 本机进程 | 服务是否启动、是否监听端口 | `ps`, `ss -tulnp`, `pidof` |
| 本机网络 | IP 地址、网卡、路由是否正常 | `ip addr`, `ip route` |
| 远端连通 | 主机是否可达、端口是否开放 | `ping`, `curl`, `nc` |
| DNS | 域名是否解析到正确地址 | `dig`, `nslookup` |
| HTTP | 状态码、响应头、请求体是否正确 | `curl -i`, `curl -v` |

一个实用顺序：

```bash
# 1. 服务进程是否存在
ps aux | grep nginx

# 2. 端口是否在监听
ss -ltnp | grep ':80'

# 3. 本机访问是否正常
curl -i http://127.0.0.1:80

# 4. 从外部地址访问是否正常
curl -i http://server.example.com
```

## 2. Ports and Connections

### 2.1 `ss`

`ss` 用于查看 socket、监听端口和连接状态。现代 Linux 系统中，它通常比 `netstat` 更推荐。

```bash
ss -tulnp
ss -ant
ss -ltnp | grep 3000
```

常用选项：

| 选项 | 含义 |
|---|---|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | 只看监听状态的 socket |
| `-n` | 不做名称解析，直接显示数字地址和端口 |
| `-p` | 显示进程信息 |
| `-a` | 显示全部 socket |

常见任务：

| 任务 | 命令 |
|---|---|
| 查看所有监听 TCP/UDP 端口 | `ss -tulnp` |
| 查看 TCP 连接 | `ss -ant` |
| 查 3000 端口是谁占用 | `ss -ltnp | grep ':3000'` |
| 查看 ESTABLISHED 连接 | `ss -ant state established` |

### 2.2 `netstat`

`netstat` 也能查看网络连接、监听端口和路由信息。很多旧资料和服务器环境仍会使用它。

| 任务 | 命令 |
|---|---|
| 查看监听 TCP/UDP 端口 | `netstat -tulnp` |
| 查看 TCP 连接，数字显示 | `netstat -ant` |
| 查 3000 端口占用 | `netstat -tulnp | grep 3000` |
| 查看路由表 | `netstat -rn` |

常用选项：

| 选项 | 含义 |
|---|---|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-p` | Show PID/program name |
| `-r` | Routing table |

### 2.3 TCP State

`ss -ant` 或 `netstat -ant` 输出里常见的 TCP 状态：

| 状态 | 含义 |
|---|---|
| `LISTEN` | 服务正在监听端口，等待连接 |
| `ESTABLISHED` | 连接已经建立，正在通信或保持连接 |
| `TIME-WAIT` | 主动关闭连接后等待旧包过期 |
| `CLOSE-WAIT` | 对端已关闭，本端程序还没有关闭 socket |
| `SYN-SENT` | 已发起连接请求，等待对端回应 |
| `SYN-RECV` | 收到连接请求，等待握手完成 |

如果 `CLOSE-WAIT` 很多，通常要怀疑应用没有正确关闭连接；如果 `TIME-WAIT` 很多，先看是否属于短连接高频请求，不一定是错误。

## 3. IP Address and Routing

### 3.1 `ip addr`

查看网卡和 IP 地址：

```bash
ip addr
ip addr show eth0
```

常见字段：

| 字段 | 含义 |
|---|---|
| `lo` | 本地回环网卡 |
| `eth0`, `ens33` | 常见物理或虚拟网卡名称 |
| `inet` | IPv4 地址 |
| `inet6` | IPv6 地址 |
| `UP` | 网卡处于启用状态 |

### 3.2 `ip route`

查看路由表：

```bash
ip route
```

典型输出：

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```

含义：

| 字段 | 含义 |
|---|---|
| `default` | 默认路由，不知道往哪走时走这里 |
| `via` | 下一跳网关 |
| `dev` | 从哪块网卡发出 |
| `src` | 本机源地址 |

## 4. Connectivity and DNS

### 4.1 `ping`

`ping` 用于测试三层连通性：

```bash
ping 8.8.8.8
ping example.com
ping -c 4 example.com
```

注意：`ping` 通不代表 TCP/HTTP 一定通；有些服务器或防火墙会禁 ICMP，所以 `ping` 不通也不一定代表业务端口不通。

### 4.2 DNS Lookup

```bash
dig example.com
nslookup example.com
```

排查域名时先看两件事：

1. 域名是否能解析出 IP；
2. 解析出的 IP 是否符合预期。

### 4.3 Port Reachability

如果系统有 `nc`：

```bash
nc -vz example.com 443
```

没有 `nc` 时，可以用 `curl` 直接测 HTTP/HTTPS 端口：

```bash
curl -I https://example.com
```

## 5. HTTP Requests with `curl`

`curl` 用于发起 HTTP 请求、下载文件和调试接口。它适合查看状态码、响应头、重定向、认证和请求体。

### 5.1 Common Options

| 选项 | 作用 |
|---|---|
| `-i` | 显示响应头和响应体 |
| `-I` | 只请求并显示响应头 |
| `-v` | 显示详细连接过程，适合调试 TLS、代理、请求头 |
| `-L` | 跟随重定向 |
| `-o file` | 保存为指定文件 |
| `-O` | 使用远端文件名保存 |
| `-H 'Key: Value'` | 添加请求头 |
| `-d 'data'` | 发送请求体，常用于 POST |
| `-X METHOD` | 指定请求方法 |
| `-u user:pass` | Basic Auth |
| `--connect-timeout N` | 设置连接超时秒数 |
| `--max-time N` | 设置整个请求最大耗时 |

### 5.2 Examples

```bash
# 查看响应头
curl -I https://example.com

# 查看响应头和响应体
curl -i https://example.com

# 跟随重定向并保存文件
curl -L -o page.html https://example.com

# JSON POST
curl -i   -H 'Content-Type: application/json'   -d '{"name":"alice"}'   https://example.com/api

# 下载 URL 列表
xargs -n 1 curl -L -O < urls.txt
```

调试接口时，先用 `-i` 看状态码和响应头；下载文件时，优先考虑 `-L`，避免只保存到重定向提示页。

### 5.3 Status Code Quick Table

| 状态码 | 含义 | 常见排查方向 |
|---|---|---|
| `200` | 请求成功 | 内容是否符合预期 |
| `301/302/307/308` | 重定向 | 是否需要 `curl -L`，Location 是否正确 |
| `400` | 请求格式错误 | 参数、JSON、Header |
| `401` | 未认证 | token、cookie、Basic Auth |
| `403` | 无权限 | 权限、IP 白名单、鉴权策略 |
| `404` | 资源不存在 | URL、路由、路径 |
| `500` | 服务端错误 | 应用日志 |
| `502/503/504` | 网关或上游异常 | 反向代理、上游服务、超时 |

## 6. `wget`

`wget` 常用于下载文件，尤其适合简单下载、断点续传和递归下载场景。

```bash
wget https://example.com/file.tar.gz
wget -O renamed.tar.gz https://example.com/file.tar.gz
wget -c https://example.com/big.iso
```

| 选项 | 作用 |
|---|---|
| `-O file` | 保存为指定文件 |
| `-c` | 断点续传 |
| `-q` | 安静模式 |
| `--limit-rate=1m` | 限速 |

## 7. Network-related `strace`

当程序“看起来像网络问题”但日志不清楚时，可以用 `strace` 看它实际调用了哪些网络系统调用：

```bash
strace -e trace=network curl http://example.com
strace -f -e trace=network ./network_program
```

常见用途：

| 场景 | 命令 |
|---|---|
| 看程序是否真的发起网络连接 | `strace -e trace=network command` |
| 跟踪子进程网络调用 | `strace -f -e trace=network command` |
| 同时看文件和网络 | `strace -e trace=file,network command` |

## 8. Troubleshooting Checklist

### 服务端口占用

```bash
ss -ltnp | grep ':3000'
ps aux | grep app
```

如果端口没有监听，优先看服务是否启动、绑定地址是否正确。服务只监听 `127.0.0.1` 时，外部机器无法直接访问。

### 域名能解析但访问失败

```bash
dig example.com
curl -v https://example.com
```

如果 DNS 正常但 HTTP 失败，继续看端口、防火墙、TLS、反向代理和应用日志。

### 本机能访问，外部不能访问

```bash
ss -ltnp
ip addr
ip route
```

重点看服务是否绑定在 `0.0.0.0` 或对应网卡 IP，而不是只绑定 `127.0.0.1`。

### 请求超时

```bash
curl --connect-timeout 3 --max-time 10 -v https://example.com
```

区分是连接阶段超时、TLS 阶段卡住，还是服务端处理太慢。
