---
title: Linux Index
tags:
  - Shell
categories:
  - Linux
date: 2026-06-24 00:00:00
katex: true
---

# Linux Index

## Notes

| 主题          | 笔记                                                 | 适合查什么                                                                          |
| ----------- | -------------------------------------------------- | ------------------------------------------------------------------------------ |
| 文件系统与基础命令   | [[Linux-Filesystem-and-Basic-Commands]]            | `pwd`、`cp`、`rm`、`cat`、`less`、`find`、`mount`、`tar`、权限、环境变量、APT/Snap             |
| 文本处理与管道     | [[Linux-Text-Processing-Pipelines]]                | `head`、`tail`、`wc`、`grep`、`sort`、`uniq`、`cut`、`tr`、`sed`、`awk`、`xargs`、`tee`   |
| 进程、作业、性能与调试 | [[Linux-Processes-Jobs-Performance-and-Debugging]] | `ps`、`top`、`jobs`、`fg`、`pidof`、`kill`、`pkill`、`time`、`strace`、`objdump`        |
| 网络与 HTTP 调试 | [[Linux-Networking-and-HTTP]]                | `ss`、`netstat`、`ip addr`、`ip route`、`ping`、`dig`、`curl`、`wget`、端口、DNS、HTTP 状态码 |

## Command Lookup

### 文件与目录

| 场景      | 命令      | 笔记                                            |
| ------- | ------- | --------------------------------------------- |
| 查看当前路径  | `pwd`   | [[Linux-Filesystem-and-Basic-Commands#pwd]]   |
| 复制      | `cp`    | [[Linux-Filesystem-and-Basic-Commands#cp]]    |
| 删除      | `rm`    | [[Linux-Filesystem-and-Basic-Commands#rm]]    |
| 查看小文件   | `cat`   | [[Linux-Filesystem-and-Basic-Commands#cat]]   |
| 分页查看大文件 | `less`  | [[Linux-Filesystem-and-Basic-Commands#less]]  |
| 查找文件    | `find`  | [[Linux-Filesystem-and-Basic-Commands#find]]  |
| 挂载      | `mount` | [[Linux-Filesystem-and-Basic-Commands#mount]] |
| 归档压缩    | `tar`   | [[Linux-Filesystem-and-Basic-Commands#tar]]   |

### 文本处理与管道

| 场景 | 命令 | 笔记 |
|---|---|---|
| 查看文件开头/末尾 | `head`, `tail` | [[Linux-Text-Processing-Pipelines#head / tail]] |
| 统计行数/词数/字节数 | `wc` | [[Linux-Text-Processing-Pipelines#wc]] |
| 搜索文本 | `grep` | [[Linux-Text-Processing-Pipelines#grep]] |
| 排序 | `sort` | [[Linux-Text-Processing-Pipelines#sort]] |
| 去除相邻重复 | `uniq` | [[Linux-Text-Processing-Pipelines#uniq]] |
| 截取字段 | `cut` | [[Linux-Text-Processing-Pipelines#cut]] |
| 字符转换/删除 | `tr` | [[Linux-Text-Processing-Pipelines#tr]] |
| 流式替换/删行 | `sed` | [[Linux-Text-Processing-Pipelines#sed]] |
| 按字段处理/聚合 | `awk` | [[Linux-Text-Processing-Pipelines#awk]] |
| 标准输入转参数 | `xargs` | [[Linux-Text-Processing-Pipelines#xargs]] |
| 输出到屏幕并写文件 | `tee` | [[Linux-Text-Processing-Pipelines#tee]] |

### 进程、性能与调试

| 场景 | 命令 | 笔记 |
|---|---|---|
| 进程快照 | `ps`, `ps aux` | [[Linux-Processes-Jobs-Performance-and-Debugging#ps]] |
| 实时资源 | `top` | [[Linux-Processes-Jobs-Performance-and-Debugging#top]] |
| PID 查询 | `pidof` | [[Linux-Processes-Jobs-Performance-and-Debugging#pidof]] |
| 终止进程 | `kill`, `pkill` | [[Linux-Processes-Jobs-Performance-and-Debugging#Signals, kill, and pkill]] |
| 作业控制 | `jobs`, `fg` | [[Linux-Processes-Jobs-Performance-and-Debugging#jobs]] |
| 耗时统计 | `time` | [[Linux-Processes-Jobs-Performance-and-Debugging#time]] |
| 系统调用追踪 | `strace` | [[Linux-Processes-Jobs-Performance-and-Debugging#strace]] |
| 二进制分析 | `objdump` | [[Linux-Processes-Jobs-Performance-and-Debugging#objdump]] |

### 网络与 HTTP

| 场景         | 命令                            | 笔记                                                             |     |
| ---------- | ----------------------------- | -------------------------------------------------------------- | --- |
| 查看监听端口     | `ss -tulnp`, `netstat -tulnp` | [[Linux-Networking-and-HTTP#2. Ports and Connections]]   |     |
| 查端口占用      | `ss -ltnp  \| grep 3000`      | [[Linux-Networking-and-HTTP#2. Ports and Connections]]   |     |
| 查看 IP 地址   | `ip addr`                     | [[Linux-Networking-and-HTTP#3. IP Address and Routing]]  |     |
| 查看路由       | `ip route`, `netstat -rn`     | [[Linux-Networking-and-HTTP#3. IP Address and Routing]]  |     |
| 测连通性       | `ping`, `nc`                  | [[Linux-Networking-and-HTTP#4. Connectivity and DNS]]    |     |
| 查 DNS      | `dig`, `nslookup`             | [[Linux-Networking-and-HTTP#4. Connectivity and DNS]]    |     |
| HTTP 请求/下载 | `curl`, `wget`                | [[Linux-Networking-and-HTTP#5. HTTP Requests with curl]] |     |
| 网络系统调用     | `strace -e trace=network`     | [[Linux-Networking-and-HTTP#7. Network-related strace]]  |     |

## Useful Pipelines

```bash
# 查找并搜索，兼容空格文件名
find . -type f -name '*.md' -print0 | xargs -0 grep -n 'pattern'

# 统计重复行
sort file.txt | uniq -c | sort -nr

# 查端口占用
ss -tulnp | grep 3000

# 后台作业回前台
jobs
fg %1
```
