---
title: Linux Processes Jobs Performance and Debugging
tags:
  - Linux
  - Shell
categories:
  - Linux
  - OS
date: 2026-06-26 00:00:00
katex: true
---

# Linux Processes, Jobs, Performance, and Debugging

## 快速索引

| 场景 | 命令 |
|---|---|
| 查看进程快照 | `ps`, `ps aux` |
| 实时查看资源 | `top` |
| 查看进程 PID | `pidof` |
| 查看命令耗时 | `time` |
| 查看/恢复 shell 作业 | `jobs`, `fg` |
| 跟踪系统调用 | `strace` |
| 查看二进制文件信息 | `objdump` |

## 关键组合

```bash
# 查看最占 CPU 的进程
ps aux --sort=-%cpu | head

# 后台作业回前台
jobs
fg %1

# 跟踪程序启动失败原因
strace -f -o trace.log ./program
```



## ps

`ps` (Process Status) 命令用于显示当前系统中的进程状态。

### 基本语法

`ps` 命令的选项有多种风格：

1.  **UNIX 风格:** 选项前需要加 `-`，可以组合 (例如 `-ef`)。
2.  **BSD 风格:** 选项前不需要加 `-`，可以组合 (例如 `aux`)。
3.  **GNU 长选项:** 选项前需要加 `--` (例如 `--forest`)。

```bash
ps [选项]
```

### 常用选项组合

*   `ps aux`: (BSD 风格) 显示所有用户的进程，包括没有控制终端的进程，并显示详细信息。
*   `ps -ef` 或 `ps -eF`: (UNIX 风格) 显示所有进程，并显示完整格式的信息 (`-F` 比 `-f` 更详细)。
*   `ps -ejH`: 显示进程树。
*   `ps axo <format>`: 自定义输出格式。

### 常用选项 (混合风格)

| 选项 (UNIX) | 选项 (BSD) | 选项 (GNU) | 描述 |
|---|---|---|---|
| `-e` | `a` (不完全等同) | `--everyone` | 显示所有进程 |
| `-f` | `u` (不完全等同) | | 显示完整格式 (UID, PID, PPID, C, STIME, TTY, TIME, CMD) |
| `-F` | | | 显示更完整的格式 (比 `-f` 多了 SZ, RSS, PSR 等) |
| `-l` | `l` | | 显示长格式 |
| `-j` | `j` | | 作业控制格式 |
| `-H` | | `--forest` | 以进程树的形式显示 |
| | `a` | | 显示所有与终端相关的进程 |
| | `x` | | 显示没有控制终端的进程 |
| | `u` | | 以用户为中心的格式显示 (USER, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME, COMMAND) |
| `-p <pid>` | `p <pid>` | `--pid <pid>` | 显示指定 PID 的进程信息 |
| `-u <user>` | `U <user>` | `--user <user>` | 显示指定用户的进程 |
| `-C <command>` | | `--command <command>` | 显示指定命令名称的进程 |
| `-o <format>` | `o <format>` | `--format <format>` | 自定义输出格式 (例如 `pid,ppid,user,%cpu,cmd`) |

### 输出列含义 (部分 `aux` 格式)

*   `USER`: 进程所有者的用户名。
*   `PID`: 进程 ID。
*   `%CPU`: 进程占用的 CPU 百分比。
*   `%MEM`: 进程占用的物理内存百分比。
*   `VSZ`: 进程使用的虚拟内存大小 (KB)。
*   `RSS`: 进程占用的物理内存大小 (KB)。
*   `TTY`: 进程关联的终端。
*   `STAT`: 进程状态 (R:运行, S:睡眠, Z:僵尸, D:不可中断睡眠, T:停止)。
*   `START`: 进程启动时间。
*   `TIME`: 进程累计使用的 CPU 时间。
*   `COMMAND`: 启动进程的命令。

### 示例

1.  **显示所有进程 (BSD 风格):**
    ```bash
    ps aux
    ```

2.  **显示所有进程 (UNIX 风格):**
    ```bash
    ps -ef
    ```

3.  **查看特定用户的进程:**
    ```bash
    ps -u username
    # 或者
    ps aux | grep '^username'
    ```

4.  **查看特定 PID 的进程:**
    ```bash
    ps -p 1234
    ```

5.  **查看进程树:**
    ```bash
    ps -ejH
    # 或者
    ps aux --forest
    ```

6.  **查找特定名称的进程 (结合 grep):**
    ```bash
    ps aux | grep 'nginx'
    ```
    *注意: 这通常也会匹配到 grep 进程本身。可以使用 `pgrep` 或 `pidof`。*

7.  **自定义输出格式:**
    ```bash
    ps -eo pid,ppid,%mem,cmd
    ```

### 注意事项

*   `ps` 命令显示的是执行瞬间的进程快照。
*   `top` 或 `htop` 命令提供动态更新的进程视图。
*   不同风格的选项有时可以混用，但推荐保持一致性以避免混淆。

## top

`top` 命令提供了一个动态的、实时的系统进程视图。它显示了系统摘要信息以及当前由 Linux 内核管理的任务列表。

### 启动 `top`

在终端中直接输入 `top` 即可启动：

```bash
top
```

### 界面概览

`top` 的输出分为两部分：

1.  **系统摘要区 (顶部):** 显示系统整体信息，如运行时间、登录用户数、负载平均值、任务总数、CPU 状态、内存使用情况、交换空间使用情况。
2.  **进程列表区 (下方):** 显示各个进程的详细信息，默认按 CPU 使用率排序。

#### 系统摘要区详解

*   **第一行:** `top - <当前时间> up <系统运行时间>, <用户数> users, load average: <1分钟>, <5分钟>, <15分钟>`
*   **第二行:** `Tasks: <总数> total, <运行中> running, <睡眠中> sleeping, <停止> stopped, <僵尸> zombie`
*   **第三行 (%Cpu(s)):** CPU 使用率
    *   `us`: 用户空间 (user)
    *   `sy`: 内核空间 (system)
    *   `ni`: 用户进程空间内改变过优先级的进程 (nice)
    *   `id`: 空闲 CPU (idle)
    *   `wa`: 等待 I/O (wait)
    *   `hi`: 硬中断 (hardware interrupt)
    *   `si`: 软中断 (software interrupt)
    *   `st`: 虚拟机偷取时间 (steal time)
*   **第四行 (KiB Mem):** 物理内存使用情况
    *   `total`: 总量
    *   `free`: 空闲
    *   `used`: 已使用
    *   `buff/cache`: 缓冲/缓存
*   **第五行 (KiB Swap):** 交换空间（虚拟内存）使用情况
    *   `total`: 总量
    *   `free`: 空闲
    *   `used`: 已使用
    *   `avail Mem`: 可用内存

#### 进程列表区详解 (部分列)

| 列名 | 描述 |
|---|---|
| `PID` | 进程 ID |
| `USER` | 进程所有者 |
| `PR` | 优先级 (Priority) |
| `NI` | Nice 值 (负值表示高优先级，正值表示低优先级) |
| `VIRT` | 虚拟内存使用量 (KB) |
| `RES` | 常驻内存使用量 (Resident Memory, KB) |
| `SHR` | 共享内存使用量 (Shared Memory, KB) |
| `S` | 进程状态 (Status: D=不可中断睡眠, R=运行, S=睡眠, T=跟踪/停止, Z=僵尸) |
| `%CPU` | CPU 使用率 |
| `%MEM` | 物理内存使用率 |
| `TIME+` | 进程使用的总 CPU 时间 (精确到百分之一秒) |
| `COMMAND` | 命令名/命令行 |

### 常用交互式命令

在 `top` 运行时，可以按以下键进行交互：

| 按键 | 功能 |
|---|---|
| `h` 或 `?` | 显示帮助信息 |
| `q` | 退出 `top` |
| `空格` 或 `Enter` | 立即刷新屏幕 |
| `d` 或 `s` | 设置刷新间隔时间 (秒) |
| `P` | 按 CPU 使用率排序 (默认) |
| `M` | 按内存使用率排序 |
| `T` | 按累计 CPU 时间排序 |
| `N` | 按 PID 排序 |
| `k` | 终止 (kill) 指定 PID 的进程 (会提示输入 PID 和信号，默认 15 SIGTERM) |
| `r` | 重新设置指定 PID 进程的 Nice 值 (会提示输入 PID 和 Nice 值) |
| `u` | 按指定用户过滤进程 (会提示输入用户名，空表示所有用户) |
| `f` | 进入字段管理界面，可以选择显示/隐藏列，以及排序列 |
| `o` 或 `O` | 进入过滤条件设置 |
| `1` | 切换显示 CPU 核心的详细信息 (多核 CPU) |
| `z` | 切换彩色/单色显示 |
| `b` | 切换高亮显示运行中的进程 |
| `c` | 切换显示命令名或完整命令行 |
| `W` | 将当前设置写入 `~/.toprc` 配置文件 |
| `L` | 查找字符串 |
| `&` | 进入分页查找模式 |

### 命令行选项 (启动时)

| 选项 | 描述 |
|---|---|
| `-d <秒数>` | 设置屏幕刷新间隔 |
| `-n <次数>` | 设置刷新次数后退出 |
| `-p <PID>` | 监控指定的 PID (最多 20 个，逗号分隔) |
| `-u <用户>` | 只显示指定用户的进程 |
| `-b` | 批处理模式，适合将输出重定向到文件或管道 |
| `-i` | 不显示空闲 (idle) 或僵尸 (zombie) 进程 |
| `-c` | 显示完整命令行而不是命令名 |

### 示例

1.  **以 5 秒间隔刷新:**
    ```bash
    top -d 5
    ```

2.  **只刷新 3 次后退出:**
    ```bash
    top -n 3
    ```

3.  **监控 PID 为 1234 和 5678 的进程:**
    ```bash
    top -p 1234,5678
    ```

4.  **只显示用户 `nginx` 的进程:**
    ```bash
    top -u nginx
    ```

5.  **将 `top` 输出保存到文件 (批处理模式):**
    ```bash
    top -b -n 1 > top_output.txt
    ```

### 注意事项

*   `top` 是一个资源密集型工具，尤其是在刷新频率很高时。
*   `htop` 是一个更现代、用户界面更友好的替代品，提供了类似的功能和更多的交互选项。
*   理解 `top` 输出的各个字段对于诊断系统性能问题非常有帮助。

## Signals

Linux 进程之间可以通过信号通信。终止进程时，优先尝试温和的 `TERM`，只有进程无响应时再考虑 `KILL`。

| 信号 | 名称 | 描述 |
|---|---|---|
| `1` | `HUP` | 挂起，常用于让守护进程重新加载配置 |
| `2` | `INT` | 中断，通常对应 `Ctrl+C` |
| `3` | `QUIT` | 退出 |
| `9` | `KILL` | 无条件终止，进程无法捕获或清理 |
| `11` | `SEGV` | 段错误 |
| `15` | `TERM` | 请求进程尽可能正常终止，默认终止信号 |
| `17` | `STOP` | 无条件停止运行，但不终止 |
| `18` | `TSTP` | 暂停并放到后台，通常对应 `Ctrl+Z` |
| `19` | `CONT` | 在 `STOP` 或 `TSTP` 后恢复执行 |

按 PID 发送信号：

```bash
kill PID
kill -15 PID
kill -9 PID
```

按进程名匹配并发送信号：

```bash
pkill nginx
pkill -f 'python app.py'
```

`kill` 更精确，适合已经知道 PID 的场景；`pkill` 更方便，但匹配范围可能比预期更大，执行前可以先用 `pgrep -a pattern` 检查匹配结果。

## jobs

`jobs` 命令用于显示当前 shell 会话中正在运行或已停止的后台作业（job）列表。

### 基本语法

```bash
jobs [选项] [%job_id ...]
```

*   `%job_id`: 可选参数，指定要显示信息的特定作业 ID。可以指定多个。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-l` | 除了显示作业 ID 和状态外，还显示进程组 ID (PGID) |
| `-p` | 只显示作业的进程组 ID (PGID) |
| `-n` | 只显示自上次通知以来状态发生变化的作业 |
| `-r` | 只显示正在运行的作业 |
| `-s` | 只显示已停止的作业 |

### 输出格式说明

`jobs` 命令的输出通常包含以下信息：

*   `[job_id]`: 作业的编号，用于 `fg`, `bg`, `kill` 等命令。
*   `+`: 当前默认作业（最近一个放入后台或停止的作业）。`fg` 或 `bg` 不带参数时操作此作业。
*   `-`: 上一个默认作业。
*   `状态`: 作业的当前状态，常见的有：
    *   `Running`: 正在后台运行。
    *   `Stopped`: 已暂停（通常由 `Ctrl+Z` 触发）。
    *   `Done`: 作业已正常完成。
    *   `Terminated`: 作业被信号终止。
*   `命令`: 启动该作业的命令。

### 示例

1.  **显示所有后台作业:**
    ```bash
    jobs
    # 输出示例:
    # [1]+  Stopped                 vim script.py
    # [2]-  Running                 sleep 60 &
    ```

2.  **显示作业及进程组 ID:**
    ```bash
    jobs -l
    # 输出示例:
    # [1]+  12345 Stopped                 vim script.py
    # [2]-  12346 Running                 sleep 60 &
    ```

3.  **只显示正在运行的作业:**
    ```bash
    jobs -r
    ```

4.  **只显示已停止的作业:**
    ```bash
    jobs -s
    ```

### 相关命令

*   `fg [%job_id]`: 将后台作业切换到前台运行。
*   `bg [%job_id]`: 让后台暂停的作业继续在后台运行。
*   `kill [%job_id | %pid]`: 向作业或进程发送信号（默认为终止信号）。
*   `Ctrl+Z`: 暂停当前前台进程，并将其放入后台。
*   `&` (命令末尾): 在后台运行命令。

### 注意事项

*   `jobs` 命令只显示当前 shell 会话启动的作业。
*   作业管理是 shell 的一项功能，不同 shell 的实现可能略有差异，但基本概念和命令（`jobs`, `fg`, `bg`）是通用的。

## fg

`fg` 命令用于将后台运行的作业（job）切换到前台继续运行。

### 基本语法

```bash
fg [%job_id]
```

*   `%job_id`: 可选参数，指定要切换到前台的作业 ID。如果不指定，则默认将最近一个放入后台的作业切换到前台。

### 如何获取 Job ID

使用 `jobs` 命令可以查看当前 shell 会话中的后台作业列表及其对应的 Job ID。

```bash
jobs
```

输出示例：
```
[1]+  Stopped                 vim my_script.py
[2]-  Running                 sleep 100 &
```

### 示例

1.  **将最近一个后台作业切换到前台:**
    ```bash
    fg
    ```

2.  **将 Job ID 为 1 的作业切换到前台:**
    ```bash
    fg %1
    ```

### 相关命令

*   `bg [%job_id]`: 将一个在后台暂停的作业，变成在后台继续执行。
*   `jobs`: 查看当前后台运行的作业。
*   `Ctrl+Z`: 将当前正在前台运行的进程暂停，并放到后台。

### 注意事项

*   `fg` 命令只能操作当前 shell 会话中的作业。
*   如果指定的 `job_id` 不存在，`fg` 会报错。
*   将作业切换到前台后，你可以像正常运行命令一样与其交互，或者使用 `Ctrl+C` 终止它。

## pidof

`pidof` 命令用于查找正在运行的程序的进程 ID (PID)。它通过程序的名称来查找。

### 基本语法

```bash
pidof [选项] <程序名...>
```

*   `<程序名>`: 要查找其 PID 的一个或多个程序名称。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-s` | 单次命中 (Single shot) - 只返回找到的第一个 PID |
| `-c` | 只返回运行在同一根目录下的进程 PID |
| `-x` | 同时查找可能正在运行的脚本的 PID |
| `-o <PID>` | 忽略具有指定 PID 的进程 |
| `-o %PPID` | 忽略调用 `pidof` 命令的父进程 |

### 示例

1.  **查找名为 `sshd` 的所有进程 PID:**
    ```bash
    pidof sshd
    # 输出示例: 1234 5678
    ```

2.  **查找名为 `bash` 的第一个进程 PID:**
    ```bash
    pidof -s bash
    # 输出示例: 9876
    ```

3.  **查找名为 `nginx` 的进程，并排除 PID 为 1000 的进程:**
    ```bash
    pidof -o 1000 nginx
    ```

4.  **查找正在运行的 `my_script.sh` 脚本的 PID:**
    ```bash
    pidof -x my_script.sh
    ```

### 返回值

*   **0**: 至少找到了一个匹配的程序。
*   **1**: 没有找到匹配的程序。

### 与 `pgrep` 的比较

*   `pidof` 通常更简单，直接通过程序名查找。
*   `pgrep` 功能更强大，允许使用正则表达式和其他条件来匹配进程。

```bash
pgrep sshd # 效果类似 pidof sshd
pgrep -u root sshd # 查找 root 用户运行的 sshd 进程
```

### 注意事项

*   `pidof` 查找的是精确的程序名称。
*   如果一个程序有多个实例在运行，`pidof` 会列出所有匹配的 PID（除非使用了 `-s` 选项）。

## time

`time` 命令用于测量程序的执行时间和资源使用情况。它可以显示程序的实际运行时间、用户CPU时间、系统CPU时间等信息。

### 基本语法

```bash
time <命令> [参数...]
/usr/bin/time [选项] <命令> [参数...]
```

*   `<命令> [参数...]`: 要测量的程序及其参数

### 两种time命令

1.  **Shell内置的time** (通常是bash内置命令)
2.  **GNU time** (通常位于 `/usr/bin/time`)

GNU time提供更多选项和详细信息。

### Shell内置time输出

```bash
time ls /tmp
##### 输出示例:
##### real    0m0.003s
##### user    0m0.001s
##### sys     0m0.002s
```

*   **real**: 实际经过的时间（墙钟时间）
*   **user**: 用户空间CPU时间
*   **sys**: 内核空间CPU时间

### GNU time 常用选项

| 选项 | 描述 |
|---|---|
| `-f <格式>`, `--format=<格式>` | 自定义输出格式 |
| `-o <文件>`, `--output=<文件>` | 将结果输出到指定文件 |
| `-a`, `--append` | 追加到输出文件而不是覆盖 |
| `-v`, `--verbose` | 显示详细信息 |
| `-p`, `--portability` | 使用POSIX格式输出 |

### GNU time 格式化选项

| 格式符 | 描述 |
|---|---|
| `%E` | 实际时间（格式：[小时:]分钟:秒） |
| `%e` | 实际时间（秒，小数格式） |
| `%U` | 用户CPU时间（秒） |
| `%S` | 系统CPU时间（秒） |
| `%P` | CPU使用百分比 ((U+S)/E) |
| `%M` | 最大常驻内存大小（KB） |
| `%t` | 平均常驻内存大小（KB） |
| `%K` | 平均总内存使用（KB） |
| `%D` | 平均非共享数据大小（KB） |
| `%p` | 平均非共享栈大小（KB） |
| `%X` | 平均共享文本大小（KB） |
| `%Z` | 系统页面大小（字节） |
| `%F` | 主要页面错误次数 |
| `%R` | 次要页面错误次数 |
| `%W` | 进程被交换出内存的次数 |
| `%c` | 进程被上下文切换的次数（非自愿） |
| `%w` | 进程主动上下文切换的次数 |
| `%I` | 文件系统输入次数 |
| `%O` | 文件系统输出次数 |
| `%r` | 接收到的套接字消息数 |
| `%s` | 发送的套接字消息数 |
| `%k` | 接收到的信号数 |
| `%x` | 程序退出状态 |
| `%C` | 被计时的命令名和命令行参数 |

### 示例

1.  **基本时间测量:**
    ```bash
    time sleep 2
    # 输出:
    # real    0m2.003s
    # user    0m0.001s
    # sys     0m0.002s
    ```

2.  **使用GNU time显示详细信息:**
    ```bash
    /usr/bin/time -v ls /tmp
    # 输出包含内存使用、页面错误等详细信息
    ```

3.  **自定义输出格式:**
    ```bash
    /usr/bin/time -f "Time: %E, Memory: %M KB" ls /tmp
    # 输出: Time: 0:00.01, Memory: 2048 KB
    ```

4.  **测量内存使用:**
    ```bash
    /usr/bin/time -f "Max Memory: %M KB" ./memory_intensive_program
    ```

5.  **将结果保存到文件:**
    ```bash
    /usr/bin/time -o timing.log -f "%E %M" ./my_program
    ```

6.  **测量编译时间:**
    ```bash
    time make
    # 或者更详细的信息
    /usr/bin/time -v make
    ```

7.  **比较不同算法的性能:**
    ```bash
    echo "Algorithm 1:"
    time ./algorithm1 input.txt
    echo "Algorithm 2:"
    time ./algorithm2 input.txt
    ```

8.  **测量CPU密集型任务:**
    ```bash
    /usr/bin/time -f "CPU: %P, Real: %E, User: %U, Sys: %S" ./cpu_intensive_task
    ```

### 时间类型解释

#### Real Time (实际时间)
*   程序从开始到结束的总时间
*   包括等待I/O、网络等的时间
*   也称为"墙钟时间"或"elapsed time"

#### User Time (用户时间)
*   程序在用户空间执行的CPU时间
*   不包括系统调用的时间
*   多核系统中可能超过real time

#### System Time (系统时间)
*   程序在内核空间执行的CPU时间
*   包括系统调用、内核操作的时间

#### CPU使用率计算

CPU使用率 = (User Time + System Time) / Real Time × 100%


### 实际应用场景

#### 1. 性能基准测试
```bash
##### 测试不同编译优化级别的效果
time gcc -O0 -o prog_O0 source.c
time gcc -O2 -o prog_O2 source.c
time gcc -O3 -o prog_O3 source.c
```

#### 2. 内存泄漏检测
```bash
##### 监控程序的内存使用
/usr/bin/time -f "Max Memory: %M KB" ./potentially_leaky_program
```

#### 3. I/O密集型vs CPU密集型分析
```bash
##### I/O密集型程序：real time >> user + sys time
time find / -name "*.txt" 2>/dev/null

##### CPU密集型程序：user time ≈ real time
time ./cpu_intensive_calculation
```

### 注意事项

*   在多核系统中，user + sys time 可能超过 real time
*   shell内置的time和GNU time的输出格式可能不同
*   对于非常短的程序，测量结果可能不够精确
*   某些shell（如zsh）可能有不同的time内置命令格式

## strace

`strace` 是一个强大的系统调用跟踪工具，用于监控和诊断程序与Linux内核之间的交互。它可以显示程序执行过程中的所有系统调用、信号和错误。

### 基本语法

```bash
strace [选项] <命令> [参数...]
strace [选项] -p <PID>
```

*   `<命令> [参数...]`: 要跟踪的程序及其参数
*   `-p <PID>`: 跟踪已运行的进程

### 常用选项

| 选项 | 描述 |
|---|---|
| `-p <PID>` | 跟踪指定进程ID的进程 |
| `-f` | 跟踪子进程（fork出的进程） |
| `-ff` | 跟踪子进程，并将每个进程的输出保存到单独的文件 |
| `-o <文件>` | 将输出重定向到指定文件 |
| `-e <表达式>` | 指定要跟踪的系统调用或事件 |
| `-c` | 统计每个系统调用的时间、调用次数和错误次数 |
| `-T` | 显示每个系统调用的耗时 |
| `-t` | 在每行前显示时间戳 |
| `-tt` | 显示微秒级时间戳 |
| `-ttt` | 显示自纪元以来的秒数 |
| `-r` | 显示相对时间戳（相对于前一个系统调用） |
| `-s <大小>` | 指定字符串的最大显示长度（默认32） |
| `-v` | 详细模式，显示完整的结构体内容 |
| `-x` | 以十六进制显示非ASCII字符串 |
| `-xx` | 以十六进制显示所有字符串 |
| `-y` | 显示文件描述符对应的文件路径 |
| `-yy` | 显示协议特定信息 |

### `-e` 表达式语法

| 表达式 | 描述 |
|---|---|
| `-e trace=<系统调用>` | 只跟踪指定的系统调用 |
| `-e trace=file` | 跟踪所有文件相关的系统调用 |
| `-e trace=process` | 跟踪所有进程相关的系统调用 |
| `-e trace=signal` | 跟踪所有信号相关的系统调用 |
| `-e trace=ipc` | 跟踪所有IPC相关的系统调用 |
| `-e trace=memory` | 跟踪所有内存相关的系统调用 |
| `-e signal=<信号>` | 跟踪指定信号 |
| `-e read=<fd>` | 跟踪指定文件描述符的读操作 |
| `-e write=<fd>` | 跟踪指定文件描述符的写操作 |

### 示例

1.  **跟踪程序的所有系统调用:**
    ```bash
    strace ls /tmp
    ```

2.  **跟踪已运行的进程:**
    ```bash
    strace -p 1234
    ```

3.  **只跟踪文件相关的系统调用:**
    ```bash
    strace -e trace=file ls /tmp
    ```

4.  **跟踪特定的系统调用:**
    ```bash
    strace -e trace=open,read,write cat /etc/passwd
    ```

5.  **统计系统调用:**
    ```bash
    strace -c ls /tmp
    # 输出类似:
    # % time     seconds  usecs/call     calls    errors syscall
    # ------ ----------- ----------- --------- --------- ----------------
    #  25.00    0.000004           2         2           read
    #  25.00    0.000004           1         4           fstat
    #   ...
    ```

6.  **显示时间戳和耗时:**
    ```bash
    strace -tt -T ls /tmp
    ```

7.  **跟踪子进程:**
    ```bash
    strace -f bash -c "ls /tmp"
    ```

8.  **将输出保存到文件:**
    ```bash
    strace -o trace.log ls /tmp
    ```


10. **显示文件路径:**
    ```bash
    strace -y cat /etc/passwd
    ```

### 常见用途

#### 1. 调试程序问题
```bash
##### 查看程序为什么无法启动
strace ./my_program

##### 查看程序访问了哪些文件
strace -e trace=file ./my_program
```

#### 2. 性能分析
```bash
##### 统计系统调用耗时
strace -c ./my_program

##### 查看哪些系统调用最耗时
strace -T ./my_program
```

#### 3. 安全审计
```bash
##### 查看程序访问了哪些文件
strace -e trace=file,process ./suspicious_program
```

#### 4. 学习系统调用
```bash
##### 了解简单命令的工作原理
strace echo "hello world"

##### 查看文件操作的系统调用
strace cp source.txt dest.txt
```

### 输出解读

典型的strace输出格式：

`system_call(arg1, arg2, ...) = return_value`

## objdump

`objdump` 命令用于显示目标文件（object files）的信息，例如可执行文件、库文件等。它常用于反汇编、查看符号表、节头信息等。

### 基本语法

```bash
objdump [选项] <目标文件...>
```

### 常用选项

| 选项 | 描述 |
|---|---|
| `-d` 或 `--disassemble` | 反汇编包含指令的代码段 |
| `-D` 或 `--disassemble-all` | 反汇编所有段 |
| `-S` 或 `--source` | 在反汇编代码中交错显示源代码（需要编译时包含调试信息，如 `-g`） |
| `-t` 或 `--syms` | 显示文件的符号表条目 |
| `-T` 或 `--dynamic-syms` | 显示文件的动态符号表条目（通常用于共享库） |
| `-h` 或 `--section-headers` | 显示文件的节头信息摘要 |
| `-x` 或 `--all-headers` | 显示所有可用头信息，包括符号表和重定位条目 |
| `-f` 或 `--file-headers` | 显示文件的整体头信息摘要 |
| `-j <section>` 或 `--section=<section>` | 只显示指定节（section）的信息 |
| `-M <dialect>` 或 `--disassembler-options=<dialect>` | 设置反汇编方言 (例如 `intel` 或 `att`) |
| `-C` 或 `--demangle[=style]` | 将低级符号名解码（demangle）为用户级名称（对 C++ 特别有用） |

### 示例

1.  **反汇编可执行文件的代码段:**
    ```bash
    objdump -d my_program
    ```

2.  **反汇编并显示对应的源代码 (需用 -g 编译):**
    ```bash
    objdump -S my_program
    ```

3.  **显示符号表:**
    ```bash
    objdump -t my_program
    ```

4.  **显示动态符号表 (例如共享库):**
    ```bash
    objdump -T /usr/lib/x86_64-linux-gnu/libc.so.6
    ```

5.  **显示所有头信息:**
    ```bash
    objdump -x my_program
    ```

6.  **反汇编 `.text` 节并使用 Intel 汇编语法:**
    ```bash
    objdump -d -j .text -M intel my_program
    ```

7.  **显示符号表并进行 C++ 名称解码:**
    ```bash
    objdump -tC my_cpp_program
    ```

### 注意事项

*   `objdump` 是 GNU Binutils 工具集的一部分。
*   它是分析二进制文件、进行底层调试和逆向工程的重要工具。
*   要获得最有用的反汇编输出（如 `-S`），需要在编译时包含调试信息 (`-g` 或 `-ggdb`)。
