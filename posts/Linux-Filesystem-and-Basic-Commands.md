---
title: Linux Filesystem and Basic Commands
tags:
  - Linux
  - Shell
categories:
  - Linux
date: 2026-06-26 00:00:00
katex: true
---

# Linux Filesystem and Basic Commands

## 快速索引

| 场景 | 命令 |
|---|---|
| 当前目录 | `pwd` |
| 复制 | `cp` |
| 删除 | `rm` |
| 查看小文件 | `cat` |
| 分页查看 | `less` |
| 查找文件 | `find` |
| 挂载存储设备 | `mount` |
| 归档压缩 | `tar` |

## Command Line Basics

这一部分记录 Linux 命令行中最容易混淆、也最常用的文件系统基础。

### Shell Basics

#### Listing Files and Directories

##### Filtering Output with Wildcards

* 模式匹配: 问号(?)代表任意单个字符; 星号(*)代表零个或多个字符。
* 方括号代表单个字符位置并给出了该位置上的多个可能的选择。你可以将可能的字符逐一列出,也可以指定字符范围,比如字母范围[a–i]。
* 还可以使用惊叹号(!)将不需要的内容排除在外。

#### File Operations

##### Copying Files

* cp 命令可以完成文件和目录从文件系统的一个位置复制到另一个位置的操作。 
* cp 命令最基本的用法需要两个参数,即源对象和目标对象:`cp source destination`。 当参数 source 和 destination 都是文件名时,cp 命令会将源文件复制成一个新的目标文件,并以 destination 命名。新文件在形式上就像全新的文件一样,有新的修改时间。
* 如果目标文件已经存在,则 cp 命令 可能并不会提醒你这一点。最好加上-i 选项,强制 shell 询问是否需要覆盖已有文件。
* cp 命令的-R 选项威力强大。可以用它在单个命令中递归地复制整个目录的内容。
* 在执行 cp –R 命令之前,目录 NewDocuments 并不存在。它是随着 cp –R 命令被创建的, 整个 Documents 目录中的内容都被复制到其中。注意,新的 NewDocuments 目录中所有的文件都 有对应的新日期。NewDocuments 目录现在已经成了 Documents 目录的完整副本。

##### Command-line Completion

* 制表键补全允许你在输入文件名 或目录名的时候,按一下制表键,让 shell 帮你自动补全内容。
* 使用制表键补全的技巧在于要给 shell 提供足够的文件名信息,使其能够将所需文件名与其 他文件名区分开。

##### Links

* 链接是目录中指 向文件真实位置的占位符。在 Linux 中有两种类型的文件链接: 符号链接，硬链接。
* 符号链接(也称为软链接)是一个实实在在的文件,该文件指向存放在虚拟目录结构中某个地方的另一个文件。这两个以符号方式链接在一起的文件彼此的内容并不相同。可以类比为windows系统中的快捷方式。
* 要为一个文件创建符号链接,原始文件必须事先存在。使用 ln -s 命令创建符号链接。`ln -s test_file slink_test_file`
* 硬链接创建的是一个独立的虚拟文件,其中包含了原始文件的信息以及位置。但是两者就根本而言是同一个文件。可以理解为“同一文件出现在不同的目录下”。要想创建硬链接,原始文件也必须事先存在,只不过这次使用 ln 命令时不需要再加入额外的选项了:`ln test_file hlink_test_file`

##### Renaming and Moving Files

* 重命名文件称为移动。mv 命令可以将文件和目录移动到另一个位置或是重新命名：`mv old new`

##### Removing Files

* rm的-i 选项会询问你是否真的要删除该文件。shell 没有回收站或者垃圾箱这样的东西, 文件一旦被删除,就再也找不回来了。所以在使用 rm 命令时,要养成总是加入-i 选项的好习惯。

#### Directories

##### Creating Directories

* 可以根据需要“批量”地创建目录和子目录。为此,要使用 mkdir 命令的-p 选项, mkdir 命令的-p 选项可以根据需要创建缺失的父目录。

#### Viewing File Content

##### File Type

* file 命令是一个方便的小工具,能够探测文件的内部并判断文件类型

##### Viewing Whole Files

1. cat

* cat 命令是显示文本文件中所有数据的得力工具。
* -n 选项会给所有的行加上行号
* 如果只想给有文本的行加上行号,可以用-b 选项

2. more

3. less

* less实为 more 命令的升级版本
* less 命令提供了多个非常实用的特性,能 够实现在文本文件中前后翻动,还有一些高级搜索功能。
* less 命令还可以在完成整个文件的读取之前显示文件的内容。

##### Viewing Parts of Files

1. tail 命令

* tail 命令会显示文件最后几行的内容(文件的“尾部”)。在默认情况下,它会显示文件的 末尾 10 行。
* 可以向 tail 命令中加入-n 选项来修改所显示的行数，如通过加入-n 2, 使得 tail 命令只显示文件的最后两行
* tail 命令有一个非常酷的特性:-f 选项,该选项允许你在其他进程使用此文件时查看文件 的内容。tail 命令会保持活动状态并持续地显示添加到文件中的内容。这是实时监测系统日志 的绝佳方式。

2. head 命令

* head 命令会显示文件开头若干行(文件的“头部”)。在默认情况下,它会显示文件前 10 行的文本
* head 命令也支持-n 选项
* 这两个命令也允 许简单地在连字符后面直接输入想要显示的行数

### Filesystem, Disk, and Archives

#### Disk and Mounted Filesystems

##### Mounting Storage Devices

* Linux 文件系统会将所有的磁盘都并入单个虚拟目录。在使用新的存储设备之前,需要将其放在虚拟目录中。这项工作称为挂载(mounting)。
* 实际的物理存储设备通过驱动被抽象成 /dev/xxx 设备文件，用户不能直接用目录树访问它们。必须把它们挂载到虚拟文件系统（VFS）的某个挂载点之后，才能以统一的方式访问和管理。

1. mount

* 用于挂载存储设备的命令叫作 mount。在默认情况下,mount 命令会输出当前系统已挂载 的设备列表。
* 如果知道设备分区使用的文件系统类型,可以使用`-t <fs-type>`过滤输出。
* mount 命令提供了 4 部分信息: 设备文件名, 设备在虚拟目录中的挂载点, 文件系统类型, 已挂载设备的访问状态
* 手动挂载设备的基本命令: `mount -t type device directory`, type 参数指定了磁盘格式化所使用的文件系统类型。示例： `mount -t vfat /dev/sdb1 /media/disk`
* -o 选项允许在挂载文件系统时添加一系列以逗号分隔的额外选项, 如:

 | 选项 | 说明 |
 |------|------|
 | ro | 以只读形式挂载 |
 | rw | 以读写形式挂载 |
 | user | 允许普通用户挂载该文件系统 |
 | check=none | 挂载文件系统时不执行完整性校验 |
 | loop | 挂载文件 |

2. umount

* 移除可移动设备时,不能直接将设备拔下,应该先卸载。
* umount 命令的格式非常简单:  `umount [directory | device ]`

##### Checking Free Space with `df`

* 有时需要知道在某台设备上还有多少磁盘空间。df 命令可以方便地查看所有已挂载磁盘的使用情况
* df 命令会输出内核挂载的所有虚拟文件系统,因此可以使用-t 选项来指定文件系统类型
* 该命令的输出: 设备文件位置, 包含多少以 1024 字节为单位的块, 使用了多少以 1024 字节为单位的块, 还有多少以 1024 字节为单位的块可用, 已用空间所占的百分比, 设备挂载点
* 常用选项之一是-h,该选项会以人类易读(humanreadable)的形式显示磁盘空间,通常用 M 来替代兆字节,用 G 来替代吉字节

##### Checking Directory Usage with `du`

* du 命令可以显示某个特定目录(默认情况下是当前 目录)的磁盘使用情况。
* 下面这些选项能让 du 命令的输出更加清晰易读。
  * -c:显示所有已列出文件的总大小。
  * -h:按人类易读格式输出大小,分别用 K 表示千字节、M 表示兆字节、G 表示吉字节。
  * -s:输出每个参数的汇总信息。

#### Compression and Archives

##### Compression with `gzip`

* gzip 命令会压缩命令行中指定的文件。也可以指定多个文件名或是用通配符来一次性压缩多个文件，压缩后的文件会在文件名后面添加 .gz 扩展名。
* 要解压缩文件,可以使用 gunzip 命令,或者使用 gzcat 命令来查看压缩文件的内容。

##### Archives with `tar`

* gzip 命令只能压缩文件，要压缩一个文件夹，需要先归档。
* 归档工具是 tar 命令`tar function [options] object1 object2 ...`
* 常用function: -c --create 创建新的 tar 归档文件 -t --list 列出 tar 归档文件的内容
* 常用option: -f file 将结果输出到文件(或设备) 
* `tar -cf test.tar test/ test2/`该命令创建了一个名为 test.tar 的归档文件, 包含目录 test 和 test2 的内容。
* `tar -tf test.tar` 该命令列出了(但不提取)tar 文件 test.tar 的内容。
* `tar -xvf test.tar` 该命令从 tar 文件 test.tar 中提取内容。如果创建的时候 tar 文件含有目录结构,则在当前目录中重建该目录的整个结构。
* 在下载开源软件时经常会看到文件名以.tgz 结尾,这是经 gzip 压缩过的 tar 文件,可以用 `tar -zxvf filename.tgz` 来提取其中的内容。

* tar 命令的操作

| 操作 | 长选项 | 描 述 |
|------|--------|-------|
| -A  | --concatenate | 将一个 tar 归档文件追加到另一个 tar 归档文件末尾 |
| -c | --create | 创建新的 tar 归档文件 |
| -d | --diff | 检查归档文件和文件系统的不同之处 |
| |--delete | 从 tar 归档文件中删除文件 |
| -r | --append | 将文件追加到 tar 归档文件末尾 |
| -t | --list | 列出 tar 归档文件的内容 |
| -u | --update | 将比 tar 归档文件中已有的同名文件更新的文件追加到该归档文件 |
| -x | --extract | 从 tar 归档文件中提取文件 |

* tar 命令选项

| 选项 | 描述 |
|------|------|
| -C dir | 切换到指定目录 |
| -f file | 将结果输出到文件(或设备) |
| -j | 将输出传给 bzip2 命令进行压缩 |
| -J | 将输出传给 xz 命令进行压缩 |
| -p | 保留文件的所有权限 |
| -v | 在处理文件时显示文件名 |
| -z | 将输出传给 gzip 命令进行压缩 |
| -Z | 将输出传给 compress 命令进行压缩 |

### Shell Commands and Environment

#### External Commands and Builtins

##### Builtin Commands

2. 使用命令别名

* alias 命令是另一个实用的 shell 内建命令。命令别名允许为常用命令及其参数创建另一个名称,从而将输入量减少到最低。`alias alias-name=origin`
* 因为命令别名属于 内建命令,所以别名仅在其被定义的 shell 进程中才有效。

### Environment Variables

#### Environment Variable Scope

* bash shell 中有两种环境变量: 全局变量, 局部变量。

##### Global Environment Variables

* 全局环境变量对于 shell 会话和所有生成的子 shell 都是可见的。局部环境变量则只对创建它 的 shell 可见。
* 可以使用 env 命令或 printenv 命令来查看全局变量
* 使用 echo 命令显示某个变量的值。在引用某个环境变量时,必须在该变量名前加上美元 符号($)

#### User-defined Variables

##### Local Shell Variables

* 使用等号为变量赋值,值可以是数值或字符串 `my_variable=Hello`
* 如果用于赋值的字符串包含空格,则必须用单引号或双引号来界定该字符串的起止
* **在变量名、等号和值之间没有空格**。如果在赋值表达式中加上了空 格,那么 bash shell 会将值视为单独的命令


##### Exported Environment Variables

* `export my_variable=value`

#### Removing Environment Variables

* 删除已有的环境变量。可以用 unset 命令来完成这 个操作。在 unset 命令中引用环境变量时,记住不要使用$。
* 如果要用到(doing anything with)变量,就使用$;如果要操作(doing anything to)变量,则不使用$。
* 对于 PATH 变量的修改只能持续到退出或重启系统。这种效果并不能一直奏效。

#### Startup Files

* 当你登录 Linux 系统启动 bash shell 时,默认情况下 bash 会在几个文件中查找命令。这些文件 称作启动文件或环境文件。

##### Login Shell Startup Files

* 登录 shell 通常会从 5 个不同的启 动文件中读取命令。  
    * /etc/profile  
    * $HOME/.bash_profile  
    * $HOME/.bashrc  
    * $HOME/.bash_login  
    * $HOME/.profile  
* /etc/profile 文件是系统中默认的 bash shell 的主启动文件。系统中的每个用户登录时都会执行这个启动文件。
* 其余的启动文件都用于同一个目的:提供用户专属的启动文件来定义该用户所用到的环境 变量。大多数 Linux 发行版只用这 4 个启动文件中的一两个。
* 在大多数发行版中,保存个人用户永久性 bash shell 变量的最佳地点是$HOME/.bashrc 文件。
* 可以把个人的 alias 设置放在$HOME/.bashrc 启动文件中,使其效果永久化。

### File Permissions

#### Reading Permission Bits

文件权限可以按下面的位段理解：

| 权限段 | 第 1 位 | 第 2 位 | 第 3 位 | 含义 |
|---|---|---|---|---|
| 属主 user | `r` | `w` | `x` | 文件拥有者的读、写、执行权限 |
| 属组 group | `r` | `w` | `x` | 文件所属组成员的读、写、执行权限 |
| 其他用户 others | `r` | `w` | `x` | 其他用户的读、写、执行权限 |

例如：

```text
-rwxrw-r--
 │ │ │
 │ │ └─ others: r--，可读，不可写，不可执行
 │ └─── group : rw-，可读写，不可执行
 └───── user  : rwx，可读写执行
```

最前面的 `-` 表示普通文件；若是目录通常为 `d`。

##### Default Permissions and `umask`

* 使用3位8进制数来表示权限。

| 权限 | 二进制值 | 八进制值 | 描述 |
|------|----------|----------|------|
| --- | 000 | 0 | 没有任何权限 |
| --x | 001 | 1 | 只有执行权限 |
| -w- | 010 | 2 | 只有写入权限 |
| -wx | 011 | 3 | 有写入和执行权限 |
| r-- | 100 | 4 | 只有读取权限 |
| r-x | 101 | 5 | 有读取和执行权限 |
| rw- | 110 | 6 | 有读取和写入权限 |
| rwx | 111 | 7 | 有全部权限(读取、写入和执行) |

* umask 命令用来设置新建文件和目录 的默认权限
* umask 值只是个掩码, 把 umask 值从对象的全权限值(full permission)中减掉。对文件而言,全权限值是 666 (所有用户都有读取和写入的权限);对目录而言,全权限值则是 777(所有用户都有读取、写入 和执行权限), 结果就是新建的文件或目录的权限。
* 文件不能直接执行。新建文件是普通数据（文本、配置、源码等），不是程序。因此全权限为666
* 必须要有 x 权限才能*进入目录*，否则你即使有 r 也只能列出文件名，看不到内容。

#### Changing Permissions

##### `chmod`

* chmod 命令可以修改文件和目录的安全设置。`chmod options mode file`
* mode 参数允许使用八进制模式（推荐）或符号模式来进行安全设置。

### Package Management

#### Package Repository Basics

* 软件包存储在称为仓库(repository)的服务器上,可以利用本地 Linux 系统中的软件包管理 器通过 Internet 访问,在其中搜索新的软件包,或是更新系统中已安装的软件包。

#### Debian/Ubuntu Packages with `apt`

* `apt [options] command`

##### Searching and Inspecting Packages

* apt list 命 令会显示仓库中所有可用的软件包,如果再加入--installed 选项,就可以限制仅输出那些已安装在系统中的软件包
* 如果已经知道系统中的某个软件包,希望显示其详细信息,可以使用 show 命令来操作: `apt show package_name`
* apt show 命令并不会指明软件包是否已经安装。它只根据软件仓库显示该软件包的详细信息。

##### Installing Packages

* 查找特定的软件包: `apt search package_name` 
* search 命令的妙处在于不需要在 package_name 周围添加通配符,直接就有通配符的效果。
* 如果只想搜索软件包名称,可以加入--name-only 选项。
* 安装：`apt install package_name `
* 可以使用 list 命令的--installed 选项检查安装是否正确。如果在输出中看到了软件包, 那么说明已经安装好了。

##### Upgrading Packages

* upgrade 命令可以使用仓库中的任何新版本安全地升级系统中*所有*的软件包:  `apt upgrade`
* upgrade 命令在升级过程中不会删除任何软件包。如果必须删除某个软件包才能完成升级, 可以使用以下命令:  `apt full-upgrade`

##### Removing Packages

* apt 的 remove 命令可以删除软件包,同时保留数据和配置文件。如果要将软件包以及相关 的数据和配置文件全部删除,那么需要使用 purge 命令
* 如果确定有依赖关系的软件包不会再有他用,可以使用 autoremove 命令将其删除

#### Application Containers

* 云计算带来了应用程序打包方式的一种新范式:应用程序容器(application container)。应用 程序容器创建了一个环境,其中包含了应用程序运行所需的全部文件,包括运行时库文件。开发 人员随后可以将应用程序容器作为单个软件包分发,保证能够在任何 Linux 系统中正常运行。

##### Snap Packages

* 使用 snap 命令行工具查询 snap 数据库,显示已安装的 snap 包,以及安装、升级和删除 snap 包。
* snap list 命令查看当前已安装的 snap 应用程序列表
* snap find 命令可以在 snap 仓库中搜索指定程序
* snap info 命令可以查看 snap 应用程序(简称为 snap)的详细信息
* snap install 命令可以安装新的 snap
* 如果需要删除某个 snap,使用 snap remove 命令即可
* Snap vs APT 的区别

| 特性       | **APT (Deb 包)**          | **Snap 包**        |
| -------- | ------------------------ | ----------------- |
| **软件形式** | `.deb` 包                 | `.snap` 包         |
| **依赖管理** | 使用系统库，依赖需要由 APT 解决       | 自带依赖，几乎无需外部依赖     |
| **安装来源** | Debian/Ubuntu 官方仓库 或 PPA | Snap Store（集中式）   |
| **升级机制** | 由系统包管理器统一升级              | Snapd 后台自动滚动更新    |
| **运行环境** | 与系统环境强耦合                 | 运行在容器化沙盒中         |
| **体积大小** | 通常较小（共用系统库）              | 较大（自带依赖）          |
| **安全性**  | 依赖系统级安全机制                | 自带沙盒隔离，更细粒度权限控制   |
| **启动速度** | 快，直接使用系统库                | 稍慢，需通过 snapd 启动环境 |
| **典型应用** | 系统级工具、传统软件               | 桌面应用、跨发行版的程序      |


## pwd

`pwd` 命令用于打印当前工作目录 (Present Working Directory) 的绝对路径。

### 基本语法

```bash
pwd [选项]
```

### 常用选项

| 选项 | 描述 |
|---|---|
| `-L` | （默认）打印 `$PWD` 环境变量的值，即使它包含符号链接。 |
| `-P` | 打印物理目录，不包含任何符号链接。 |

### 示例

1.  **打印当前工作目录:**
    ```bash
    pwd
    # 输出示例: /home/user/documents
    ```

2.  **如果当前目录是通过符号链接访问的，打印物理路径:**
    假设 `/home/user/link_to_docs` 是指向 `/usr/local/share/documents` 的符号链接。
    ```bash
    cd /home/user/link_to_docs
    pwd -L
    # 输出: /home/user/link_to_docs
    pwd -P
    # 输出: /usr/local/share/documents
    ```

### 返回值

*   **0**: 成功执行。
*   **非 0**: 发生错误。

### 注意事项

*   `pwd` 是一个非常基础且常用的命令，用于在命令行中确定您当前所在的位置。
*   大多数 Shell（如 Bash）将 `pwd` 作为内置命令实现，以提高效率。

## cat

`cat` 命令用于连接文件并打印到标准输出设备上。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-n` 或 `--number` | 对所有输出行编号 |
| `-b` 或 `--number-nonblank` | 对非空输出行编号 |
| `-s` 或 `--squeeze-blank` | 当遇到有连续两行以上的空白行，就代换为一行的空白行 |
| `-E` 或 `--show-ends` | 在每行结束处显示 `$` |
| `-T` 或 `--show-tabs` | 将 TAB 字符显示为 `^I` |
| `-A` 或 `--show-all` | 等同于 `-vET` |
| `-v` 或 `--show-nonprinting` | 使用 `^` 和 `M-` 符号，除了 LFD 和 TAB 之外 |

### 示例

1.  **显示文件内容:**
    ```bash
    cat file.txt
    ```

2.  **显示文件内容并显示行号:**
    ```bash
    cat -n file.txt
    ```

3.  **合并多个文件内容:**
    ```bash
    cat file1.txt file2.txt > combined_file.txt
    ```

4.  **创建文件 (使用 EOF):**
    ```bash
    cat > new_file.txt << EOF
    这是第一行。
    这是第二行。
    EOF
    ```

5.  **追加内容到文件:**
    ```bash
    cat >> existing_file.txt << EOF
    这是追加的内容。
    EOF
    ```

### 注意事项

*   对于大型文件，使用 `less` 或 `more` 命令查看通常更合适，因为 `cat` 会一次性将所有内容加载到内存中。
*   使用重定向 `>` 会覆盖目标文件内容，而 `>>` 会追加内容。

## less

`less` 命令是一个强大的分页器，用于交互式地查看文本文件内容，特别适用于大文件，因为它不会一次性将整个文件加载到内存中。

### 基本语法

```bash
less [选项] [文件名]
```

### 常用选项

| 选项 | 描述 |
|---|---|
| `-N` | 显示行号 |
| `-S` | 禁止长行自动换行（水平滚动） |
| `-i` | 搜索时忽略大小写 |
| `-M` | 显示更详细的提示信息（百分比、行号等） |
| `-X` | 禁止 `less` 在退出时清屏 |
| `-F` | 如果文件内容能在一屏内显示完，则直接输出并退出 |
| `+<行号>` | 从指定行号开始显示 |
| `+/ <模式>` | 从第一个匹配模式的位置开始显示 |

### 常用内部命令 (在 less 界面中)

| 按键 | 描述 |
|---|---|
| `空格键` 或 `f` 或 `Ctrl+F` | 向下翻一页 |
| `b` 或 `Ctrl+B` | 向上翻一页 |
| `d` 或 `Ctrl+D` | 向下翻半页 |
| `u` 或 `Ctrl+U` | 向上翻半页 |
| `j` 或 `Enter` | 向下移动一行 |
| `k` | 向上移动一行 |
| `G` | 跳转到文件末尾 |
| `g` | 跳转到文件开头 |
| `<数字>g` | 跳转到指定行号 |
| `/ <模式>` | 向下搜索指定的模式 |
| `? <模式>` | 向上搜索指定的模式 |
| `n` | 重复上一次搜索（同方向） |
| `N` | 重复上一次搜索（反方向） |
| `h` | 显示帮助信息 |
| `q` | 退出 `less` |
| `v` | 使用环境变量 `$EDITOR` 指定的编辑器打开当前文件 |
| `& <模式>` | 只显示匹配模式的行 |

### 示例

1.  **查看文件:**
    ```bash
    less large_log_file.log
    ```

2.  **查看文件并显示行号:**
    ```bash
    less -N system.log
    ```

3.  **从管道输入查看:**
    ```bash
    dmesg | less
    ```

4.  **搜索文件中的特定文本:**
    打开文件后，输入 `/error` 并按 Enter 键。

5.  **从第 100 行开始查看:**
    ```bash
    less +100 config.txt
    ```

### 注意事项

*   `less` 比 `more` 命令更强大，允许向前和向后滚动。
*   它是查看日志文件和长文本输出的常用工具。

## cp

`cp` 命令用于复制文件或目录。

### 基本语法

```bash
cp [选项] <源文件或目录> <目标文件或目录>
```

### 常用选项

| 选项 | 描述 |
|---|---|
| `-r` 或 `-R` 或 `--recursive` | 递归复制目录及其内容 |
| `-i` 或 `--interactive` | 覆盖前询问 |
| `-f` 或 `--force` | 强制覆盖，不询问 |
| `-p` 或 `--preserve` | 保留源文件的属性（包括所有者、组、权限、时间戳） |
| `-a` 或 `--archive` | 等同于 `-dR --preserve=all`，常用于归档和备份 |
| `-v` 或 `--verbose` | 显示详细的复制过程 |
| `-u` 或 `--update` | 仅当源文件比目标文件新，或者目标文件不存在时才复制 |
| `-l` 或 `--link` | 创建硬链接，而不是复制文件 |
| `-s` 或 `--symbolic-link` | 创建符号链接，而不是复制文件 |
| `-T` 或 `--no-target-directory` | 将目标视为普通文件而不是目录 |
| `--backup[=CONTROL]` | 为每个已存在的目标文件创建备份 |

### 示例

1.  **复制文件到当前目录:**
    ```bash
    cp /path/to/source/file.txt .
    ```

2.  **复制文件并重命名:**
    ```bash
    cp source.txt destination.txt
    ```

3.  **递归复制目录:**
    ```bash
    cp -r /path/to/source_directory /path/to/destination_directory
    ```

4.  **复制时保留文件属性:**
    ```bash
    cp -p source.txt destination.txt
    ```

5.  **复制时强制覆盖:**
    ```bash
    cp -f source.txt destination.txt
    ```

6.  **交互式复制 (覆盖前询问):**
    ```bash
    cp -i source.txt destination.txt
    ```

7.  **复制多个文件到目录:**
    ```bash
    cp file1.txt file2.txt /path/to/directory/
    ```

### 注意事项

*   默认情况下，`cp` 复制文件时，新文件的所有者是执行命令的用户。
*   使用 `-a` 选项通常是复制目录的最佳实践，因为它能尽可能保留原始属性。
*   目标路径如果是一个已存在的目录，源文件/目录会被复制到该目录下。如果目标路径不存在，则会创建该文件/目录。

## rm

`rm` 命令用于删除文件或目录。

### 基本语法

```bash
rm [选项] <文件/目录...>
```

*   `<文件/目录...>`: 要删除的一个或多个文件或目录的名称。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-f`, `--force` | 强制删除，忽略不存在的文件，从不提示。 |
| `-i` | 在每次删除前提示确认。 |
| `-I` | 在删除超过三个文件或者递归删除前提示一次。此选项比 `-i` 干扰少，但仍然提供对大多数错误的保护。 |
| `-r`, `-R`, `--recursive` | 递归地删除目录及其内容。 **使用此选项时务必小心！** |
| `-d`, `--dir` | 删除空目录。 |
| `-v`, `--verbose` | 显示详细的删除过程。 |

### 示例

1.  **删除名为 `myfile.txt` 的文件:**
    ```bash
    rm myfile.txt
    ```

2.  **删除多个文件:**
    ```bash
    rm file1.txt file2.log image.jpg
    ```

3.  **删除文件前提示确认:**
    ```bash
    rm -i important_document.doc
    # 会提示: rm: remove regular file 'important_document.doc'? 输入 y 确认, n 取消
    ```

4.  **强制删除文件，不提示:**
    ```bash
    rm -f temp_file
    ```

5.  **递归删除名为 `mydir` 的目录及其所有内容 (危险操作):**
    ```bash
    rm -r mydir
    # 或
    rm -rf mydir # 强制递归删除，极其危险，请谨慎使用！
    ```

6.  **删除一个空目录 `emptydir`:**
    ```bash
    rm -d emptydir
    # 注意: 如果目录非空，此命令会失败。通常使用 rmdir 命令删除空目录。
    ```

7.  **删除时显示详细信息:**
    ```bash
    rm -v old_file.txt
    # 输出示例: removed 'old_file.txt'
    ```

### 返回值

*   **0**: 成功执行。
*   **非 0**: 发生错误。

### 注意事项

*   **`rm` 命令删除的文件通常无法恢复！** 在使用特别是 `-r` 或 `-f` 选项时，请务必确认您要删除的是正确的文件或目录。
*   避免使用 `rm -rf /` 或类似的命令，这可能导致系统严重损坏。
*   对于重要的文件，建议先备份再删除。
*   `rmdir` 命令专门用于删除空目录，比 `rm -d` 更常用。

## find

`find` 命令用于在目录树中搜索文件和目录。它是Linux系统中最强大和最常用的搜索工具之一。

### 基本语法

```bash
find [搜索路径] [搜索条件] [动作]
```

*   `[搜索路径]`: 指定搜索的起始目录。如果省略，默认为当前目录 (`.`)。
*   `[搜索条件]`: 指定搜索的条件，如文件名、类型、大小等。
*   `[动作]`: 对找到的文件执行的操作。如果省略，默认为 `-print`（打印文件路径）。

### 常用搜索条件

#### 按名称搜索

| 选项 | 描述 |
|---|---|
| `-name <模式>` | 按文件名搜索（区分大小写，支持通配符） |
| `-iname <模式>` | 按文件名搜索（不区分大小写，支持通配符） |
| `-path <模式>` | 按完整路径搜索（支持通配符） |
| `-ipath <模式>` | 按完整路径搜索（不区分大小写，支持通配符） |

#### 按类型搜索

| 选项 | 描述 |
|---|---|
| `-type f` | 普通文件 |
| `-type d` | 目录 |
| `-type l` | 符号链接 |
| `-type b` | 块设备文件 |
| `-type c` | 字符设备文件 |
| `-type p` | 命名管道 (FIFO) |
| `-type s` | 套接字 |

#### 按大小搜索

| 选项 | 描述 |
|---|---|
| `-size +<大小>` | 大于指定大小 |
| `-size -<大小>` | 小于指定大小 |
| `-size <大小>` | 等于指定大小 |
| `-empty` | 空文件或空目录 |

大小单位：`c`(字节), `k`(KB), `M`(MB), `G`(GB)

#### 按时间搜索

| 选项 | 描述 |
|---|---|
| `-mtime +<天数>` | 修改时间超过指定天数 |
| `-mtime -<天数>` | 修改时间在指定天数内 |
| `-atime +<天数>` | 访问时间超过指定天数 |
| `-ctime +<天数>` | 状态改变时间超过指定天数 |
| `-newer <文件>` | 比指定文件更新 |

#### 按权限搜索

| 选项 | 描述 |
|---|---|
| `-perm <权限>` | 精确匹配权限 |
| `-perm -<权限>` | 包含指定权限 |
| `-perm /<权限>` | 任意匹配指定权限 |

### 常用动作

| 动作 | 描述 |
|---|---|
| `-print` | 打印文件路径（默认动作） |
| `-print0` | 打印文件路径，以null字符分隔（适合与xargs -0配合） |
| `-ls` | 以ls -dils格式列出文件 |
| `-delete` | 删除找到的文件 **（危险操作，请谨慎使用）** |
| `-exec <命令> {} \;` | 对每个找到的文件执行命令 |
| `-exec <命令> {} +` | 对找到的文件批量执行命令 |
| `-ok <命令> {} \;` | 交互式执行命令（每次都询问） |

### 逻辑操作符

| 操作符 | 描述 |
|---|---|
| `-and` 或 `-a` | 逻辑与（默认） |
| `-or` 或 `-o` | 逻辑或 |
| `-not` 或 `!` | 逻辑非 |
| `( )` | 分组（需要转义：`\( \)`） |

### 示例

1.  **查找当前目录下所有 `.txt` 文件:**
    ```bash
    find . -name "*.txt"
    ```

2.  **查找 `/home` 目录下所有大于 100MB 的文件:**
    ```bash
    find /home -size +100M
    ```

3.  **查找 7 天内修改过的文件:**
    ```bash
    find . -mtime -7
    ```

4.  **查找空目录:**
    ```bash
    find . -type d -empty
    ```

5.  **查找并删除所有 `.tmp` 文件:**
    ```bash
    find . -name "*.tmp" -delete
    ```

6.  **查找文件并执行命令:**
    ```bash
    find . -name "*.log" -exec gzip {} \;
    ```

7.  **查找具有特定权限的文件:**
    ```bash
    find . -perm 755
    ```

8.  **复杂条件搜索（查找 .c 或 .h 文件，但排除 test 目录）:**
    ```bash
    find . \( -name "*.c" -o -name "*.h" \) -not -path "*/test/*"
    ```

### 注意事项

*   使用 `-delete` 动作时要格外小心，建议先用 `-print` 确认要删除的文件。
*   在使用通配符时，记得用引号括起来，避免shell展开。
*   `find` 命令会递归搜索，对于大目录可能会比较慢。
*   可以使用 `-maxdepth` 限制搜索深度，使用 `-mindepth` 设置最小搜索深度。

## mount

`mount` 命令用于将文件系统挂载到Linux目录树中的指定位置。它是Linux系统管理中的重要命令，用于访问存储设备、网络文件系统等。

### 基本语法

```bash
mount [选项] <设备> <挂载点>
mount [选项] <设备>
mount [选项] <挂载点>
mount [选项]
```

*   `<设备>`: 要挂载的设备文件（如 `/dev/sdb1`）或网络路径
*   `<挂载点>`: 挂载到的目录路径

### 常用选项

| 选项 | 描述 |
|---|---|
| `-t <类型>`, `--types <类型>` | 指定文件系统类型（ext4, ntfs, vfat, nfs等） |
| `-o <选项>`, `--options <选项>` | 指定挂载选项（多个选项用逗号分隔） |
| `-r`, `--read-only` | 以只读方式挂载 |
| `-w`, `--rw`, `--read-write` | 以读写方式挂载（默认） |
| `-a`, `--all` | 挂载 `/etc/fstab` 中的所有文件系统 |
| `-v`, `--verbose` | 显示详细信息 |
| `-f`, `--fake` | 模拟挂载（不实际执行） |
| `-n`, `--no-mtab` | 不写入 `/etc/mtab` |
| `-l`, `--show-labels` | 显示文件系统标签 |
| `-U <UUID>` | 通过UUID挂载设备 |
| `-L <标签>` | 通过标签挂载设备 |

### 常用挂载选项 (`-o`)

| 选项 | 描述 |
|---|---|
| `rw` | 读写模式（默认） |
| `ro` | 只读模式 |
| `exec` | 允许执行二进制文件（默认） |
| `noexec` | 禁止执行二进制文件 |
| `suid` | 允许suid和sgid位生效（默认） |
| `nosuid` | 忽略suid和sgid位 |
| `dev` | 解释设备文件（默认） |
| `nodev` | 不解释设备文件 |
| `auto` | 可以用 `-a` 选项自动挂载 |
| `noauto` | 不能用 `-a` 选项自动挂载 |
| `user` | 允许普通用户挂载 |
| `nouser` | 只允许root挂载（默认） |
| `owner` | 允许设备所有者挂载 |
| `sync` | 同步I/O |
| `async` | 异步I/O（默认） |
| `remount` | 重新挂载已挂载的文件系统 |
| `loop` | 挂载回环设备（如ISO文件） |
| `bind` | 绑定挂载（将目录挂载到另一个位置） |

### 常见文件系统类型

| 类型 | 描述 |
|---|---|
| `ext4` | Linux默认文件系统 |
| `ext3` | 较老的Linux文件系统 |
| `xfs` | 高性能文件系统 |
| `btrfs` | 现代Linux文件系统 |
| `ntfs` | Windows NTFS文件系统 |
| `vfat` | FAT32文件系统 |
| `exfat` | exFAT文件系统 |
| `iso9660` | CD/DVD文件系统 |
| `nfs` | 网络文件系统 |
| `cifs` | Windows网络共享 |
| `tmpfs` | 内存文件系统 |

### 示例

#### 基本挂载操作

1.  **挂载USB设备:**
    ```bash
    sudo mount /dev/sdb1 /mnt/usb
    ```

2.  **指定文件系统类型挂载:**
    ```bash
    sudo mount -t ntfs /dev/sdb1 /mnt/windows
    ```

3.  **以只读方式挂载:**
    ```bash
    sudo mount -r /dev/sdb1 /mnt/readonly
    # 或者
    sudo mount -o ro /dev/sdb1 /mnt/readonly
    ```

4.  **查看当前挂载的文件系统:**
    ```bash
    mount
    # 或者只显示特定类型
    mount -t ext4
    ```

#### 高级挂载操作

5.  **挂载ISO文件:**
    ```bash
    sudo mount -o loop /path/to/image.iso /mnt/iso
    ```

6.  **绑定挂载（将目录挂载到另一个位置）:**
    ```bash
    sudo mount --bind /home/user/documents /mnt/docs
    ```

7.  **挂载网络文件系统 (NFS):**
    ```bash
    sudo mount -t nfs server:/path/to/share /mnt/nfs
    ```

8.  **挂载Windows共享 (CIFS):**
    ```bash
    sudo mount -t cifs //server/share /mnt/windows -o username=user,password=pass
    ```

9.  **创建内存文件系统:**
    ```bash
    sudo mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk
    ```

10. **通过UUID挂载:**
    ```bash
    sudo mount -U 12345678-1234-1234-1234-123456789abc /mnt/disk
    ```

11. **重新挂载文件系统（改变挂载选项）:**
    ```bash
    sudo mount -o remount,ro /mnt/disk
    ```

#### 实用组合

12. **挂载所有fstab中的文件系统:**
    ```bash
    sudo mount -a
    ```

13. **显示挂载信息（包括标签）:**
    ```bash
    mount -l
    ```

### 相关文件

#### `/etc/fstab`
系统启动时自动挂载的文件系统配置文件。格式：

<设备> <挂载点> <文件系统类型> <选项>

示例：

/dev/sda1 / ext4 defaults 0 1
/dev/sda2 /home ext4 defaults 0 2
UUID=12345678-1234-1234-1234-123456789abc /data ext4 defaults 0 2

#### `/proc/mounts`
当前挂载的文件系统信息（内核视图）

#### `/etc/mtab`
当前挂载的文件系统信息（用户空间视图）

### 卸载文件系统

使用 `umount` 命令卸载：
```bash
#### 通过挂载点卸载
sudo umount /mnt/usb

#### 通过设备卸载
sudo umount /dev/sdb1

#### 强制卸载（谨慎使用）
sudo umount -f /mnt/usb

#### 懒卸载（当不再使用时自动卸载）
sudo umount -l /mnt/usb
```

### 故障排除

#### 常见错误及解决方法

1.  **"mount: unknown filesystem type"**
    - 安装相应的文件系统支持包
    - 检查内核模块是否加载

2.  **"mount: /dev/sdb1 is already mounted"**
    - 先卸载再重新挂载
    - 或使用 `remount` 选项

3.  **"mount: /mnt/usb: mount point does not exist"**
    - 创建挂载点目录：`sudo mkdir -p /mnt/usb`

4.  **权限问题**
    - 使用 `sudo` 执行挂载命令
    - 检查 `/etc/fstab` 中的 `user` 选项

#### 查看挂载状态
```bash
#### 查看所有挂载点
mount | column -t

#### 查看特定设备
mount | grep sdb1

#### 查看文件系统使用情况
df -h

#### 查看设备信息
lsblk
```

### 注意事项

*   挂载前确保挂载点目录存在
*   卸载前确保没有进程在使用该文件系统
*   对于可移动设备，使用前先挂载，使用后及时卸载
*   修改 `/etc/fstab` 前先备份
*   网络文件系统挂载需要网络连接正常

## tar

`tar` (Tape Archive) 命令用于创建、查看、提取归档文件（通常称为 tarball 或 tar 文件）。它经常与压缩工具（如 `gzip` 或 `bzip2`）结合使用，以创建压缩归档。

### 基本语法

```bash
tar [选项] [归档文件名] [文件/目录...]
```

### 主要操作模式 (必须指定一个)

| 模式 | 描述 |
|---|---|
| `-c`, `--create` | 创建一个新的归档文件。 |
| `-x`, `--extract`, `--get` | 从归档文件中提取文件。 |
| `-t`, `--list` | 列出归档文件的内容。 |
| `-r`, `--append` | 向现有归档文件的末尾追加文件（不适用于压缩归档）。 |
| `-u`, `--update` | 仅当文件比归档中的对应文件新时，才将其追加到归档末尾（不适用于压缩归档）。 |

### 常用选项

| 选项 | 描述 |
|---|---|
| `-f <归档文件名>`, `--file=<归档文件名>` | 指定要操作的归档文件名。如果为 `-`，则表示标准输入/输出。 **通常是必需的。** |
| `-v`, `--verbose` | 显示详细的操作过程，列出正在处理的文件。 |
| `-z`, `--gzip` | 通过 `gzip` 处理归档文件（创建 `.tar.gz` 或 `.tgz` 文件）。 |
| `-j`, `--bzip2` | 通过 `bzip2` 处理归档文件（创建 `.tar.bz2` 文件）。 |
| `-J`, `--xz` | 通过 `xz` 处理归档文件（创建 `.tar.xz` 文件）。 |
| `-C <目录>`, `--directory=<目录>` | 在提取文件前，先切换到指定的目录。 |
| `-p`, `--preserve-permissions` | 在提取文件时保留文件的原始权限。 |
| `--exclude=<模式>` | 在创建归档时排除匹配模式的文件或目录。 |
| `--strip-components=<数量>` | 在提取时去除指定数量的前导目录层级。 |

### 示例

1.  **创建未压缩的归档文件 `archive.tar`，包含 `file1`, `file2`, `dir1`:**
    ```bash
    tar -cvf archive.tar file1 file2 dir1
    ```
    *   `c`: 创建
    *   `v`: 显示过程
    *   `f`: 指定文件名

2.  **创建 `gzip` 压缩的归档文件 `archive.tar.gz`:**
    ```bash
    tar -czvf archive.tar.gz file1 file2 dir1
    ```
    *   `z`: 使用 gzip 压缩

3.  **创建 `bzip2` 压缩的归档文件 `archive.tar.bz2`:**
    ```bash
    tar -cjvf archive.tar.bz2 file1 file2 dir1
    ```
    *   `j`: 使用 bzip2 压缩

4.  **列出 `archive.tar.gz` 的内容:**
    ```bash
    tar -tzvf archive.tar.gz
    ```
    *   `t`: 列出内容
    *   `z`: 处理 gzip 压缩

5.  **从 `archive.tar.gz` 提取所有文件到当前目录:**
    ```bash
    tar -xzvf archive.tar.gz
    ```
    *   `x`: 提取
    *   `z`: 处理 gzip 压缩

6.  **从 `archive.tar.bz2` 提取文件到 `/tmp/extract_here` 目录:**
    ```bash
    tar -xjvf archive.tar.bz2 -C /tmp/extract_here
    ```
    *   `x`: 提取
    *   `j`: 处理 bzip2 压缩
    *   `-C`: 指定提取目录

7.  **创建归档，但排除所有 `.log` 文件:**
    ```bash
    tar -czvf backup.tar.gz --exclude='*.log' /path/to/source
    ```

8.  **提取时去除第一层目录:**
    假设 `archive.tar.gz` 包含 `somedir/file1`, `somedir/file2`。
    ```bash
    tar -xzvf archive.tar.gz --strip-components=1
    # 这将在当前目录创建 file1 和 file2，而不是 somedir/file1 和 somedir/file2
    ```

### 返回值

*   **0**: 成功执行。
*   **非 0**: 发生错误。

### 注意事项

*   选项前的 `-` 通常可以省略（例如 `tar cvf` 等同于 `tar -cvf`），但这被认为是不推荐的风格。
*   压缩选项 (`-z`, `-j`, `-J`) 不能与追加 (`-r`) 或更新 (`-u`) 操作一起使用。
*   确保有足够的磁盘空间来创建或提取归档文件。
*   文件名参数 (`-f`) 通常放在选项之后，或者作为最后一个选项参数。
