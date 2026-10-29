---
title: Remote File Transfer
tags:
  - SSH
  - Rsync
categories:
  - Tool
date: 2026-06-24 00:00:00
katex: true
---

# 1. 使用 scp 传文件

## 1.1. 上传文件：把本地文件送到远程

语法：`scp [选项] 本地文件 user@example.com:远程路径`

端口用大写 `-P`，写成小写 `-p` 会被 scp 当成保留时间戳的选项，结果连到 22 端口并报连接失败。远程路径末尾带斜杠表示放进该目录，不带斜杠且该路径不存在时会被当成新文件名。默认覆盖同名文件不询问，路径写错时可能静默覆盖目标端文件。

```bash
# 非默认端口必须用大写 P
scp -P 2222 ./report.txt user@example.com:/srv/reports/
```

## 1.2. 下载与递归：取回文件或整个目录

语法：`scp -r [选项] user@example.com:远程路径 本地路径`

取回单个文件把两边顺序对调即可；传目录必须加 `-r`，漏写只会报 `not a regular file`。scp 每次都是完整传输，中断后重传整个文件，也不做增量比较，不适合大文件或需要反复同步的目录。

```bash
# 下载单个文件到当前目录
scp user@example.com:/srv/reports/report.txt ./
# 递归下载整个目录
scp -r user@example.com:/srv/reports/ ./reports/
```

# 2. 使用 rsync 同步

## 2.1. 同步目录：递归和属性要自己开

语法：`rsync [选项] 源 目标`

rsync 默认只处理文件列表里的文件，同步目录必须自己加 `-r` 递归，或加 `-a`。`-a` 等价于 `-rlptgoD`，除递归外还保留权限、时间戳，代价是需要相应的权限和目标端支持；`-v` 打印传输列表，`-P` 等于 `--partial --progress`，保留部分传输的文件并显示进度。

```bash
# -a 递归并保留属性，-v 显示详情，-P 显示进度且支持续传
rsync -avP ./dir/ user@example.com:/srv/dir/
```

## 2.2. 源路径尾部斜杠：决定同步内容还是整个目录

语法：`rsync -avP 源/ 目标`

源路径以斜杠结尾表示同步目录里的内容，不带斜杠表示把目录本身放进目标位置。两种写法都合法，差别只在目标端多不多一层目录名。

```bash
# 结果是 /srv/dir/file.txt
rsync -avP ./dir/ user@example.com:/srv/dir/
# 结果是 /srv/dir/dir/file.txt
rsync -avP ./dir user@example.com:/srv/dir/
```

## 2.3. 删除目标端多余文件：先模拟执行

语法：`rsync -avP --delete 源 目标`

`--delete` 让目标端与源端完全一致，会删除目标端源端没有的文件与目录，运行方向写反就可能清空目标端。先用 `--dry-run` 走一遍，它只打印会发生的传输与删除，不真正复制、覆盖或删除，确认输出无误后再去掉该选项执行。

```bash
# 模拟执行，检查输出中是否出现不该删的路径
rsync -avP --delete --dry-run ./dir/ user@example.com:/srv/dir/
# 确认后再真正执行
rsync -avP --delete ./dir/ user@example.com:/srv/dir/
```

## 2.4. 指定 ssh 端口：rsync 不认 -p

语法：`rsync -avP -e "ssh -p 端口" 源 目标`

rsync 自己用 `-e` 指定远程 shell，端口要写在传给 ssh 的字符串里；写成 `-e "ssh -P 2222"` 会被 ssh 当成未知选项而失败。`~/.ssh/config` 里已配好端口时，别名可以直接用，无需 `-e`；别名与密钥的配置见 [[SSH]]。

```bash
# 端口写在 ssh 的子命令里
rsync -avP -e "ssh -p 2222" ./dir/ user@example.com:/srv/dir/
```

# 3. 选择 scp 还是 rsync

传输规模与频率决定工具：`scp` 只做一次完整传输，`rsync` 会比较两端并只传差异。

| 场景 | 选择 |
|---|---|
| 临时传一个小文件 | `scp`，无需额外选项 |
| 传目录、大文件、需要重复同步 | `rsync`，且必须自加 `-a` 或 `-r` |
| 断点续传与进度显示 | `rsync -P`，scp 没有对应功能 |
| 要求目标端与源端完全一致 | `rsync --delete`，执行前先加 `--dry-run` |
