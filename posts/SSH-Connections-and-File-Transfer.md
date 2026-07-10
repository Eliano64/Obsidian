---
title: SSH Connections and File Transfer
tags:
  - SSH
categories:
  - SSH
date: 2026-06-24 00:00:00
katex: true
---

# SSH Connections and File Transfer

本文记录 SSH 登录、密钥认证、端口转发，以及 `scp` / `rsync` 远程传文件。

## 1. 基本连接

```bash
ssh username@host
ssh -p 2222 username@host
ssh username@192.168.1.10
```

远程执行一条命令：

```bash
ssh user@host "CMD"
ssh user@host "uname -a && uptime"
```

常用调试：

```bash
ssh -v username@host
```

`-v` 会输出详细连接过程，适合排查认证失败、密钥不匹配、配置文件未生效等问题。

## 2. 密钥认证

生成密钥：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

也可以使用更现代的 Ed25519：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

复制公钥到远程服务器：

```bash
ssh-copy-id username@hostname
```

指定私钥连接：

```bash
ssh -i ~/.ssh/id_rsa username@hostname
```

常见权限要求：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

如果权限过宽，SSH 可能拒绝使用私钥。

## 3. SSH 配置文件

在 `~/.ssh/config` 中配置常用服务器：

```sshconfig
Host myserver
    HostName example.com
    User myusername
    Port 2222
    IdentityFile ~/.ssh/my_key
```

之后可以直接：

```bash
ssh myserver
scp file.txt myserver:/tmp/
rsync -avP ./dir/ myserver:/path/on/server/
```

## 4. 实用连接选项

| 命令 | 作用 |
|---|---|
| `ssh -X user@host` | 启用 X11 转发，运行远程图形界面程序 |
| `ssh -C user@host` | 启用压缩，弱网下可能有用 |
| `ssh -o ServerAliveInterval=60 user@host` | 定期发送心跳，保持连接活跃 |
| `ssh -J jumphost user@targethost` | 通过跳板机连接目标机器 |

## 5. 端口转发

### 本地端口转发

```bash
ssh -L 8080:localhost:80 username@hostname
```

含义：访问本机 `localhost:8080`，流量会通过 SSH 隧道转发到远程机器看到的 `localhost:80`。

### 远程端口转发

```bash
ssh -R 8080:localhost:80 username@hostname
```

含义：远程机器上的 `8080` 端口转发回本机的 `80` 端口。

### 动态端口转发

```bash
ssh -D 1080 username@hostname
```

创建 SOCKS 代理，常用于浏览器或其他工具临时走 SSH 隧道。

后台建立隧道，不打开交互 shell：

```bash
ssh -f -N -L 8080:localhost:80 username@hostname
```

## 6. 使用 scp 传文件

上传单个文件：

```bash
scp local_file username@remote_ip:/path/on/server/
```

指定端口：

```bash
scp -P port local_file username@remote_ip:/path/on/server/
```

递归上传目录：

```bash
scp -r local_dir username@remote_ip:/path/on/server/
```

从远程下载：

```bash
scp username@remote_ip:/path/on/server/file ./
```

如果需要强制密码认证，原碎片里的写法可以保留：

```bash
scp -P port -o PreferredAuthentications=password -o PubkeyAuthentication=no xxx username@remote_ip:/path/on/server/xxx
```

但日常更推荐配置密钥认证，安全性和体验都更好。

## 7. 使用 rsync 传文件

基础上传：

```bash
rsync -avP xxx username@remote_ip:/path/on/server/
```

常用选项：

| 选项 | 含义 |
|---|---|
| `-a` | archive 模式，保留权限、时间等信息，适合同步目录 |
| `-v` | 输出详细信息 |
| `-P` | 等价于 `--partial --progress`，显示进度并支持保留部分传输文件 |
| `-h` | 人类可读的大小显示 |
| `--delete` | 删除目标端多余文件，使目标与源一致；慎用 |

同步目录时注意尾部斜杠：

```bash
rsync -avP ./dir/ user@host:/remote/dir/
```

`./dir/` 表示同步目录内容；`./dir` 表示把整个 `dir` 目录放到目标位置。

使用 SSH 指定端口：

```bash
rsync -avP -e "ssh -p 2222" ./dir/ user@host:/remote/dir/
```

## 8. 选 scp 还是 rsync？

| 场景 | 推荐 |
|---|---|
| 临时传一个小文件 | `scp` |
| 传目录、大文件、重复同步 | `rsync` |
| 需要断点续传/进度显示 | `rsync -P` |
| 需要保持目标与源一致 | `rsync --delete`，但先 `--dry-run` |


因为 `--delete` 会删除目标端多余文件。先 `--dry-run` 看一遍，即**先模拟执行一次同步，但不真的复制、覆盖、删除任何文件**。确认没问题后，再去掉 `--dry-run` 真正执行：

```bash
rsync -avP --delete ./dir/ user@host:/remote/dir/
```
