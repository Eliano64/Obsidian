---
title: SSH
tags:
  - SSH
categories:
  - Tool
date: 2026-06-24 00:00:00
katex: true
---

# 1. 连接与远程执行

## 1.1. 登录远程主机：建立交互式会话

语法：`ssh [选项] user@example.com`

省略端口时 ssh 连目标主机的 22 端口，换端口用 `-p`。首次连接会显示目标主机指纹并询问是否继续，确认后写入 `~/.ssh/known_hosts`。目标主机换过密钥后 ssh 报 `REMOTE HOST IDENTIFICATION HAS CHANGED` 并拒绝连接，这不是网络故障：先用带外方式核对新指纹，再从 `known_hosts` 删掉旧条目。

```bash
# 默认端口，进入交互式登录 shell
ssh user@example.com
# 换端口。ssh 的端口选项是小写 p，scp 是大写 P，两者不能混用
ssh -p 2222 user@example.com
```

## 1.2. 在远程执行命令：默认不分配终端

语法：`ssh user@example.com "命令"`

命令由远程登录 shell 执行，本地 shell 先处理引号、`$变量` 和通配符，本地会展开的内容要用单引号包住或转义。默认不分配终端，`sudo` 要密码、`top`、`vim` 这类程序会失败，`sudo` 的报错是 `a terminal is required to read the password`。此时加 `-t` 强制分配伪终端，不需要终端的命令加了只会多一层伪终端。

```bash
# 取回结果后立即断开，适合远程检查
ssh user@example.com "uname -a && uptime"
# -t 分配伪终端，让远程 sudo 能读密码
ssh -t user@example.com "sudo systemctl restart nginx"
```

## 1.3. 排查连接过程：打印握手与认证细节

语法：`ssh -v user@example.com`

`-v` 的输出写到标准错误，可叠加为 `-vv`、`-vvv` 增加细节，级别越高越难读，通常 `-v` 足够。判断认证问题时看两行：`Authentications that can continue:` 列出服务器允许的方式，`Permission denied (publickey)` 表示没有一把私钥被接受。

```bash
# 日志单独落盘，避免与远程命令的输出混在一起
ssh -v user@example.com true 2> ssh-debug.log
```

# 2. 密钥认证

## 2.1. 生成密钥对：得到私钥和公钥

语法：`ssh-keygen -t ed25519 -C "注释"`

优先选 ed25519：密钥短、签名快、没有位数需要调。`-b` 的位数只对 RSA 有意义，`-t rsa -b 4096` 只在目标服务器过旧、不认识 ed25519 时才需要。`-C` 只是写进公钥末尾的注释，可省略。默认写到 `~/.ssh/id_ed25519` 和 `~/.ssh/id_ed25519.pub`，同名文件存在时会询问是否覆盖，误覆盖会让已经部署该公钥的服务器失去免密登录能力。私钥与公钥成对出现，只有 `.pub` 可以外传。

```bash
# -f 指定路径，避免覆盖已有密钥
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_work -C "work key"
```

## 2.2. 上传公钥：让服务器认可你的私钥

语法：`ssh-copy-id -i ~/.ssh/id_ed25519.pub user@example.com`

这一步需要先用密码登录一次，它把公钥追加到目标账号的 `~/.ssh/authorized_keys`。服务器端 `~/.ssh` 必须是 700、`authorized_keys` 必须是 600，否则 sshd 忽略该文件，现象是一直索要密码。端口不是 22 时用 `-p`，漏写会连到 22 端口并报连接超时。

```bash
# 指定端口与要上传的公钥，结束前会要求输入一次密码
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519.pub user@example.com
```

## 2.3. 指定私钥连接：本机存在多个身份

语法：`ssh -i ~/.ssh/私钥 user@example.com`

`-i` 指向私钥文件，公钥不需要在这里指定。私钥权限过宽时 ssh 直接拒绝，报错含 `UNPROTECTED PRIVATE KEY FILE` 或 `Permissions ... are too open`：`~/.ssh` 用 700，私钥用 600，公钥可以 644。家目录本身不能让组或其他用户可写，否则同一项检查也会失败。

```bash
# 修正权限后连接
chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_ed25519
ssh -i ~/.ssh/id_ed25519 user@example.com
```

# 3. 主机别名

## 3.1. 用 ~/.ssh/config 固化常用主机

语法：`Host 别名` 后跟缩进的配置项

ssh 只读 `~/.ssh/config`，不需要其他注册步骤，`ssh`、`scp`、`rsync` 都会解析它。`HostName` 是真实地址，`User`、`Port`、`IdentityFile` 提供连接参数。多个 `Host` 块命中同一主机时按先出现者取值，所以通配的 `Host *` 要放在文件末尾。块名只是匹配模式，`Host myserver` 不会自动补全域名。

```text
# ~/.ssh/config
Host myserver
    HostName example.com
    User user
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_work
# 兜底项写在最后，避免覆盖具名主机的设置
Host *
    ServerAliveInterval 60
```

```bash
# 别名对三种工具同时生效
ssh myserver
rsync -avP ./dir/ myserver:/srv/dir/
```

# 4. 常用连接选项

这些选项可叠加在 `ssh user@example.com` 之前。`-X` 要求远程 sshd 允许 X11 转发且本机有 X 显示服务，转发图形界面明显变慢；`-C` 在压缩收益低于链路开销时反而更慢。

| 选项 | 含义 |
|---|---|
| `-X` | 启用 X11 转发，运行远程图形程序 |
| `-C` | 请求压缩，弱网下可能更快 |
| `-o ServerAliveInterval=60` | 每 60 秒发一次心跳，减少空闲连接被中断 |
| `-J jumphost` | 经跳板机连接目标主机，本地无需直连目标机 |
| `-p 2222` | 指定目标端口，只有 ssh 用这个写法 |

# 5. 端口转发

## 5.1. 本地端口转发：把本机端口接到远端服务

语法：`ssh -L [绑定地址:]本机端口:目标主机:目标端口 user@example.com`

流量从本机绑定端口进入隧道，再由远程主机发起连接到 `目标主机:目标端口`，`目标主机` 是远程主机视角的地址。默认只绑定 `127.0.0.1`，同网段的其他机器访问不到；要让它们可用，必须把绑定地址写成 `0.0.0.0`，这同时把服务暴露给同网段。

```bash
# 访问本机 8080 等于访问远程主机看到的 db.internal:5432
ssh -L 8080:db.internal:5432 user@example.com
# 0.0.0.0 才对本机之外开放
ssh -L 0.0.0.0:8080:db.internal:5432 user@example.com
```

## 5.2. 远程端口转发：把远端端口接回本机服务

语法：`ssh -R [绑定地址:]远程端口:目标主机:目标端口 user@example.com`

流量从远程主机的绑定端口进入隧道，再由本机发起连接到 `目标主机:目标端口`。默认只在远程主机的回环地址上监听，远程主机之外无法使用；要对外监听必须在远程 sshd 配置里设 `GatewayPorts yes` 并重启 sshd，否则写 `0.0.0.0` 也不生效。

```bash
# 在远程主机上访问 localhost:8080 等于访问本机的 3000
ssh -R 8080:localhost:3000 user@example.com
```

## 5.3. 动态端口转发：在本机开一个 SOCKS 代理

语法：`ssh -D [绑定地址:]本机端口 user@example.com`

`-D` 建立的不是到某个固定服务的隧道，而是 SOCKS 代理，端口按 SOCKS5 协议解析目标地址。默认监听 `127.0.0.1:1080`，需要其他机器使用时才写绑定地址，写 `0.0.0.0` 相当于把代理开放给同网段。

```bash
# 浏览器或命令行工具把 SOCKS5 代理指向 127.0.0.1:1080
ssh -D 1080 user@example.com
```

## 5.4. 后台隧道：不占用终端的转发

语法：`ssh -f -N -L 本机端口:目标主机:目标端口 user@example.com`

`-N` 表示不执行远程命令，`-f` 表示认证完成后转入后台，两者通常一起用。转后台前认证提示仍会占住终端，所以要用密钥或 `~/.ssh/config` 配好的身份；`-f` 之后报错不再回显到终端，隧道是否建立要自己检查。关闭隧道用 `kill` 结束对应进程。

```bash
# 建立后台隧道，随后用 ss 确认本机端口已监听
ssh -f -N -L 8080:db.internal:5432 user@example.com
```

传文件（`scp` 与 `rsync`）见 [[../Remote-File-Transfer]]。
