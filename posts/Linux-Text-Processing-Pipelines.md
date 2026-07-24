---
title: Linux Text Processing Pipelines
tags:
  - Shell
categories:
  - Linux
date: 2026-06-26 00:00:00
katex: true
---

# Linux Text Processing and Pipelines

## 常见组合

```bash
# 在一批文件中搜索文本，兼容空格文件名
find . -type f -name '*.md' -print0 | xargs -0 grep -n 'pattern'

# 排序后统计重复项
sort access.log | uniq -c | sort -nr

# 看日志最后 100 行，并继续追踪新增内容
tail -n 100 -f app.log

# 统计行数、单词数、字节数
wc -l -w -c file.txt

# 取 CSV 的第 1、3 列
cut -d ',' -f 1,3 data.csv

# 批量替换文本
sed 's/old/new/g' file.txt

# 按字段聚合统计
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log
```

## 文本处理工具地图

| 类别 | 命令 | 作用 | 典型用法 |
|---|---|---|---|
| 查看片段 | `head` | 查看文件开头 | `head -n 20 file.log` |
| 查看片段 | `tail` | 查看文件末尾 / 追踪新增 | `tail -n 100 -f app.log` |
| 统计 | `wc` | 统计行数、词数、字节数 | `wc -l file.txt` |
| 搜索 | `grep` | 按模式筛选行 | `grep -n "ERROR" app.log` |
| 排序 | `sort` | 按行排序 | `sort -k2n data.txt` |
| 去重 | `uniq` | 合并相邻重复行 | `sort file | uniq -c` |
| 切列 | `cut` | 按分隔符取字段 | `cut -d ':' -f 1 /etc/passwd` |
| 字符转换 | `tr` | 替换/删除字符 | `tr 'a-z' 'A-Z'` |
| 流编辑 | `sed` | 替换、删除、按行处理 | `sed 's/foo/bar/g' file` |
| 字段处理 | `awk` | 按字段计算/聚合 | `awk '{print $1}' file` |
| 参数转换 | `xargs` | stdin 转命令参数 | `find . -print0 | xargs -0 rm` |
| 输出分流 | `tee` | 同时输出到屏幕和文件 | `cmd | tee out.log` |

## head / tail


`head` 和 `tail` 适合快速查看文件的一部分，尤其是日志文件。

### head：查看文件开头

默认显示前 10 行：

```bash
head file.txt
```

指定行数：

```bash
head -n 20 file.txt
```

### tail：查看文件末尾

默认显示最后 10 行：

```bash
tail file.txt
```

指定行数：

```bash
tail -n 50 app.log
```

实时追踪新增内容，常用于看日志：

```bash
tail -f app.log
```

组合：先看最后 100 行，再持续追踪：

```bash
tail -n 100 -f app.log
```

## wc


`wc` 用于统计文本数量。

| 选项 | 含义 |
|---|---|
| `-l` | 行数 |
| `-w` | 单词数 |
| `-c` | 字节数 |
| `-m` | 字符数 |

```bash
wc file.txt
wc -l file.txt
grep "ERROR" app.log | wc -l
```

## cut

`cut` 用于按字符、字节或分隔字段截取文本。最常用的是按分隔符取列。

```bash
# 取 /etc/passwd 中的用户名列
cut -d ':' -f 1 /etc/passwd

# 取 CSV 第 1、3 列
cut -d ',' -f 1,3 data.csv
```

常用选项：

| 选项 | 含义 |
|---|---|
| `-d` | 指定字段分隔符 |
| `-f` | 指定字段编号，从 1 开始 |
| `-c` | 按字符位置截取 |

## tr

`tr` 用于字符级转换、删除或压缩重复字符。它从标准输入读取，不直接接文件名。

```bash
# 小写转大写
echo "hello" | tr 'a-z' 'A-Z'

# 删除数字
echo "a1b2c3" | tr -d '0-9'

# 压缩连续空格为一个空格
echo "a    b" | tr -s ' '
```

## sed

`sed` 是流编辑器，适合做按行替换、删除、打印等操作。

```bash
# 每行第一个 old 替换为 new
sed 's/old/new/' file.txt

# 每行所有 old 替换为 new
sed 's/old/new/g' file.txt

# 删除空行
sed '/^$/d' file.txt

# 只打印第 10 到 20 行
sed -n '10,20p' file.txt
```

如果要直接修改文件，GNU sed 常用：

```bash
sed -i 's/old/new/g' file.txt
```

注意：`sed -i` 会直接修改文件，重要文件先备份。

## awk

`awk` 更像“按行读取、按字段处理”的小语言，适合列处理、统计和聚合。

默认按空白分隔字段：

```bash
awk '{print $1}' file.txt
```

指定分隔符：

```bash
awk -F ':' '{print $1}' /etc/passwd
```

求和：

```bash
awk '{sum += $2} END {print sum}' data.txt
```

按第一列计数：

```bash
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log
```

在日志处理中，常见链路是：

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

含义：取第一列，例如 IP；排序；统计重复次数；按次数降序；取前几名。

## tee

`tee` 用于把管道输出一边显示在终端，一边写入文件。

```bash
make 2>&1 | tee build.log
```

追加写入：

```bash
command | tee -a run.log
```

它适合记录命令执行日志，同时保留屏幕输出。

## grep

`grep` (Global Regular Expression Print) 命令用于在文件中搜索包含指定模式的行。

### 基本语法

```bash
grep [选项] <模式> [文件...]
```

*   `<模式>`: 要搜索的文本模式（可以是普通字符串或正则表达式）。
*   `[文件...]`: 要搜索的文件列表。如果省略，`grep` 会从标准输入读取。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-i` 或 `--ignore-case` | 忽略大小写 |
| `-v` 或 `--invert-match` | 显示不匹配模式的行 |
| `-n` 或 `--line-number` | 显示匹配行的行号 |
| `-c` 或 `--count` | 只显示匹配行的数量 |
| `-l` 或 `--files-with-matches` | 只显示包含匹配行的文件名 |
| `-L` 或 `--files-without-match` | 只显示不包含匹配行的文件名 |
| `-r` 或 `-R` 或 `--recursive` | 递归搜索目录下的文件 |
| `-w` 或 `--word-regexp` | 只匹配整个单词 |
| `-E` 或 `--extended-regexp` | 使用扩展正则表达式 (ERE) |
| `-F` 或 `--fixed-strings` | 将模式视为固定字符串（禁用正则表达式） |
| `-A <num>` 或 `--after-context=<num>` | 显示匹配行及其后的 `num` 行 |
| `-B <num>` 或 `--before-context=<num>` | 显示匹配行及其前的 `num` 行 |
| `-C <num>` 或 `--context=<num>` | 显示匹配行及其前后各 `num` 行 |
| `--color[=WHEN]` | 对匹配的文本进行颜色高亮 (WHEN 可以是 `never`, `always`, `auto`) |

### 示例

1.  **在文件中搜索字符串:**
    ```bash
    grep "error" logfile.txt
    ```

2.  **忽略大小写搜索:**
    ```bash
    grep -i "warning" system.log
    ```

3.  **递归搜索目录:**
    ```bash
    grep -r "config" /etc/
    ```

4.  **显示匹配行的行号:**
    ```bash
    grep -n "main" program.c
    ```

5.  **显示不匹配的行:**
    ```bash
    grep -v "^#" config.conf  # 排除注释行
    ```

6.  **统计匹配行的数量:**
    ```bash
    grep -c "processed" data.csv
    ```

7.  **搜索包含特定单词的行:**
    ```bash
    grep -w "function" script.sh
    ```

8.  **使用扩展正则表达式:**
    ```bash
    grep -E "(error|fail)" messages
    ```

9.  **从管道输入中搜索:**
    ```bash
    ps aux | grep "nginx"
    ```

### 正则表达式元字符 (部分)

*   `^`: 匹配行首
*   `$`: 匹配行尾
*   `.`: 匹配任意单个字符
*   `*`: 匹配前一个字符零次或多次
*   `+`: 匹配前一个字符一次或多次 (需配合 `-E`)
*   `?`: 匹配前一个字符零次或一次 (需配合 `-E`)
*   `[]`: 匹配方括号内的任意一个字符 (例如 `[abc]`)
*   `[^]`: 匹配不在方括号内的任意一个字符 (例如 `[^0-9]`)
*   `\`: 转义特殊字符

### 注意事项

*   当模式包含空格或特殊字符时，建议使用引号将其括起来。
*   `grep` 是 Linux 系统中最常用的文本处理工具之一，熟练掌握其用法对系统管理和开发非常有帮助。

## sort

`sort` 命令用于对文本文件的行进行排序。它可以按字母顺序、数字顺序或自定义规则对文件内容进行排序。

### 基本语法

```bash
sort [选项] [文件...]
```

*   `[文件...]`: 要排序的文件。如果省略或为 `-`，则从标准输入读取。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-r`, `--reverse` | 逆序排序（降序） |
| `-n`, `--numeric-sort` | 按数值排序 |
| `-h`, `--human-numeric-sort` | 按人类可读的数值排序（如 1K, 2M, 3G） |
| `-f`, `--ignore-case` | 忽略大小写 |
| `-u`, `--unique` | 去除重复行，只保留唯一行 |
| `-k <字段>`, `--key=<字段>` | 按指定字段排序 |
| `-t <分隔符>`, `--field-separator=<分隔符>` | 指定字段分隔符 |
| `-o <文件>`, `--output=<文件>` | 将结果输出到指定文件 |
| `-c`, `--check` | 检查文件是否已排序，不进行排序 |
| `-m`, `--merge` | 合并已排序的文件 |
| `-s`, `--stable` | 稳定排序（相等元素保持原有顺序） |
| `-b`, `--ignore-leading-blanks` | 忽略行首的空白字符 |
| `-d`, `--dictionary-order` | 字典序排序（只考虑字母、数字和空格） |
| `-g`, `--general-numeric-sort` | 按通用数值排序（支持科学计数法） |
| `-M`, `--month-sort` | 按月份排序（Jan, Feb, Mar...） |
| `-R`, `--random-sort` | 随机排序 |
| `-V`, `--version-sort` | 按版本号排序 |

### 字段指定格式

`-k` 选项的格式：`-k <开始字段>[,<结束字段>][<选项>]`

*   字段编号从 1 开始
*   可以指定字段内的字符位置：`-k 1.2,1.5`（第1字段的第2到第5个字符）
*   可以为特定字段指定排序选项：`-k 2n`（第2字段按数值排序）

### 示例

假设有文件 `data.txt` 内容如下：

```txt
banana 3 10.5
apple 1 5.2
cherry 10 15.8
apple 2 7.3
```

1.  **基本排序（按字母顺序）:**
    ```bash
    sort data.txt
    # 输出:
    # apple 1 5.2
    # apple 2 7.3
    # banana 3 10.5
    # cherry 10 15.8
    ```

2.  **逆序排序:**
    ```bash
    sort -r data.txt
    # 输出:
    # cherry 10 15.8
    # banana 3 10.5
    # apple 2 7.3
    # apple 1 5.2
    ```

3.  **按第二字段数值排序:**
    ```bash
    sort -k 2n data.txt
    # 输出:
    # apple 1 5.2
    # apple 2 7.3
    # banana 3 10.5
    # cherry 10 15.8
    ```

4.  **按第三字段数值排序（降序）:**
    ```bash
    sort -k 3nr data.txt
    # 输出:
    # cherry 10 15.8
    # banana 3 10.5
    # apple 2 7.3
    # apple 1 5.2
    ```

5.  **多字段排序（先按第一字段，再按第二字段数值排序）:**
    ```bash
    sort -k 1,1 -k 2n data.txt
    # 输出:
    # apple 1 5.2
    # apple 2 7.3
    # banana 3 10.5
    # cherry 10 15.8
    ```

6.  **去除重复行:**
    ```bash
    sort -u data.txt
    ```

7.  **按文件大小排序（人类可读格式）:**
    ```bash
    ls -lh | sort -k 5h
    ```

8.  **排序CSV文件（按第二列）:**
    ```bash
    sort -t ',' -k 2 file.csv
    ```

### 常用组合

1.  **统计文件中最常出现的行:**
    ```bash
    sort file.txt | uniq -c | sort -nr
    ```

2.  **按IP地址排序:**
    ```bash
    sort -t . -k 1,1n -k 2,2n -k 3,3n -k 4,4n ip_list.txt
    ```

3.  **按日期排序（YYYY-MM-DD格式）:**
    ```bash
    sort -k 1 dates.txt
    ```

### 注意事项

*   默认情况下，`sort` 按字典序排序，数字 "10" 会排在 "2" 前面。使用 `-n` 选项进行数值排序。
*   `sort` 命令默认使用整行进行比较，除非指定了 `-k` 选项。
*   对于大文件，`sort` 会自动使用临时文件进行外部排序。
*   稳定排序 (`-s`) 在多字段排序时很有用，可以保持相等元素的原有顺序。

## uniq

`uniq` 命令用于报告或删除文件中的重复行。它通常与 `sort` 命令配合使用，因为 `uniq` 只能检测相邻的重复行。

### 基本语法

```bash
uniq [选项] [输入文件] [输出文件]
```

*   `[输入文件]`: 要处理的文件。如果省略或为 `-`，则从标准输入读取。
*   `[输出文件]`: 输出文件。如果省略，则输出到标准输出。

### 常用选项

| 选项 | 描述 |
|---|---|
| `-c`, `--count` | 在每行前显示该行出现的次数 |
| `-d`, `--repeated` | 只显示重复的行（出现次数 > 1） |
| `-D`, `--all-repeated[=METHOD]` | 显示所有重复的行，包括重复行的所有副本 |
| `-u`, `--unique` | 只显示唯一的行（出现次数 = 1） |
| `-i`, `--ignore-case` | 忽略大小写 |
| `-f <N>`, `--skip-fields=<N>` | 跳过前N个字段 |
| `-s <N>`, `--skip-chars=<N>` | 跳过前N个字符 |
| `-w <N>`, `--check-chars=<N>` | 只比较每行的前N个字符 |
| `-z`, `--zero-terminated` | 行以null字符结尾而不是换行符 |

### `-D` 选项的METHOD值

| METHOD | 描述 |
|---|---|
| `none` | 不分组（默认） |
| `prepend` | 在每组前插入空行 |
| `separate` | 在组之间插入空行 |

### 示例

假设有文件 `data.txt` 内容如下：

```txt
apple
banana
apple
cherry
banana
banana
date
```


#### 基本用法

1.  **删除重复行（需要先排序）:**
    ```bash
    sort data.txt | uniq
    # 输出:
    # apple
    # banana
    # cherry
    # date
    ```

2.  **统计每行出现次数:**
    ```bash
    sort data.txt | uniq -c
    # 输出:
    #       2 apple
    #       3 banana
    #       1 cherry
    #       1 date
    ```

3.  **只显示重复的行:**
    ```bash
    sort data.txt | uniq -d
    # 输出:
    # apple
    # banana
    ```

4.  **只显示唯一的行（不重复的行）:**
    ```bash
    sort data.txt | uniq -u
    # 输出:
    # cherry
    # date
    ```

5.  **显示所有重复行的副本:**
    ```bash
    sort data.txt | uniq -D
    # 输出:
    # apple
    # apple
    # banana
    # banana
    # banana
    ```

#### 高级用法

6.  **忽略大小写:**
    ```bash
    # 假设文件包含 Apple 和 apple
    sort data.txt | uniq -i
    ```

7.  **跳过字段进行比较:**
    假设文件 `users.txt` 内容：
    ```
    1 john smith
    2 john doe
    3 jane smith
    4 john smith
    ```
    
    跳过第一个字段（ID），按姓名去重：
    ```bash
    sort users.txt | uniq -f 1
    # 输出:
    # 2 john doe
    # 1 john smith
    # 3 jane smith
    ```

8.  **只比较前N个字符:**
    ```bash
    # 只比较前3个字符
    sort data.txt | uniq -w 3
    ```

### 常用组合命令

#### 1. 统计最常出现的行
```bash
sort file.txt | uniq -c | sort -nr
# 按出现频率降序排列
```

#### 2. 找出文件中的重复行及其出现次数
```bash
sort file.txt | uniq -c | awk '$1 > 1'
# 或者
sort file.txt | uniq -cd
```

#### 3. 比较两个文件的差异（简单版本）
```bash
# 找出file1中有但file2中没有的行
sort file1.txt file2.txt file2.txt | uniq -u
```

#### 4. 删除文件中的重复行（保持原有顺序）
```bash
# 使用awk保持原有顺序
awk '!seen[$0]++' file.txt

# 或者使用 uniq（会改变顺序）
sort file.txt | uniq
```

#### 5. 统计日志文件中的唯一IP地址
```bash
# 假设IP在第一列
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

### 实际应用场景

#### 1. 日志分析
```bash
# 统计访问最多的IP
cut -d' ' -f1 access.log | sort | uniq -c | sort -nr | head -10

# 统计HTTP状态码分布
awk '{print $9}' access.log | sort | uniq -c
```

#### 2. 数据清理
```bash
# 清理邮件列表中的重复邮箱
sort email_list.txt | uniq > clean_email_list.txt

# 统计重复邮箱数量
sort email_list.txt | uniq -d | wc -l
```

#### 3. 系统监控
```bash
# 统计进程名称
ps aux | awk '{print $11}' | sort | uniq -c | sort -nr

# 统计登录用户
who | awk '{print $1}' | sort | uniq
```

### 注意事项

*   **`uniq` 只检测相邻的重复行**，因此通常需要先使用 `sort` 排序
*   如果要保持原有顺序去重，考虑使用 `awk '!seen[$0]++'`
*   对于大文件，`sort | uniq` 的组合可能比较慢，可以考虑其他工具如 `awk` 或 `python`
*   字段分隔符默认是空白字符（空格和制表符）

## xargs

### 基本语法
```bash
xargs [选项] [命令 [初始参数]]
```

### 功能描述
xargs 从标准输入读取数据，将其转换为命令行参数，并执行指定的命令。它主要用于将管道输出转换为其他命令的参数。

### 常用选项

| 选项 | 描述 |
|------|------|
| `-0, --null` | 输入项以 null 字符分隔而不是空白字符 |
| `-a file` | 从文件读取输入而不是标准输入 |
| `-d delim` | 指定输入分隔符 |
| `-I replace-str` | 用输入项替换指定字符串 |
| `-i[replace-str]` | 与 -I 类似，默认替换字符串为 {} |
| `-L max-lines` | 每次最多使用指定行数的输入 |
| `-l[max-lines]` | 与 -L 类似，默认为 1 行 |
| `-n max-args` | 每次最多使用指定数量的参数 |
| `-p, --interactive` | 执行前提示确认 |
| `-P max-procs` | 最多同时运行指定数量的进程 |
| `-r, --no-run-if-empty` | 如果输入为空则不运行命令 |
| `-s max-chars` | 限制命令行的最大字符数 |
| `-t, --verbose` | 执行前打印命令 |
| `-x, --exit` | 如果超过大小限制则退出 |

### 基本用法示例

#### 1. 基本用法
```bash
# 将 find 输出作为 rm 的参数
find . -name "*.tmp" | xargs rm

# 等价于手动执行
rm file1.tmp file2.tmp file3.tmp
```

#### 2. 指定参数数量
```bash
# 每次只传递一个参数
echo "1 2 3 4 5" | xargs -n 1 echo
# 输出：
## 1
## 2
## 3
## 4
## 5

# 每次传递两个参数
echo "1 2 3 4 5" | xargs -n 2 echo
# 输出：
## 1 2
## 3 4
## 5
```

#### 3. 使用替换字符串
```bash
# 使用 -I 指定替换字符串
find . -name "*.txt" | xargs -I {} cp {} /backup/

# 使用 -i 默认替换字符串 {}
find . -name "*.txt" | xargs -i cp {} {}.bak
```

#### 4. 处理包含空格的文件名
```bash
# 使用 -0 处理 null 分隔的输入
find . -name "*.txt" -print0 | xargs -0 rm

# 使用 -d 指定分隔符
echo "file1.txt,file2.txt,file3.txt" | xargs -d ',' rm
```

#### 5. 并行执行
```bash
# 同时运行 4 个进程
find . -name "*.jpg" | xargs -P 4 -I {} convert {} {}.png

# 处理大量文件时的并行压缩
find . -name "*.log" | xargs -P 8 -I {} gzip {}
```

#### 6. 交互式确认
```bash
# 删除前确认
find . -name "*.tmp" | xargs -p rm

# 显示将要执行的命令
find . -name "*.txt" | xargs -t wc -l
```

### 实际应用场景

#### 1. 文件操作
```bash
# 批量重命名文件
ls *.txt | xargs -I {} mv {} {}.backup

# 批量修改文件权限
find . -name "*.sh" | xargs chmod +x

# 批量复制文件
find /source -name "*.conf" | xargs -I {} cp {} /destination/
```

#### 2. 文本处理
```bash
# 在多个文件中搜索
find . -name "*.log" | xargs grep "ERROR"

# 统计多个文件的行数
find . -name "*.c" | xargs wc -l

# 批量替换文件内容
find . -name "*.txt" | xargs sed -i 's/old/new/g'
```

#### 3. 系统管理
```bash
# 批量杀死进程
ps aux | grep "python" | awk '{print $2}' | xargs kill

# 批量创建目录
echo "dir1 dir2 dir3" | xargs mkdir
```

#### 4. 数据库操作
```bash
# 批量导入 SQL 文件
find . -name "*.sql" | xargs -I {} mysql -u user -p database < {}

# 批量备份数据库
echo "db1 db2 db3" | xargs -I {} mysqldump {} > {}.sql
```

### 与其他命令的组合

#### 1. 与 find 组合
```bash
# 查找并删除空文件
find . -type f -empty | xargs rm

# 查找并压缩大文件
find . -size +100M | xargs gzip
```

#### 2. 与 grep 组合
```bash
# 在搜索结果中进一步处理
grep -l "pattern" *.txt | xargs sed -i 's/old/new/g'
```

#### 3. 与 cut/awk 组合
```bash
# 提取特定字段并处理
ps aux | awk '{print $2}' | xargs -I {} kill -9 {}
```

### 注意事项

1. **空格和特殊字符**：处理包含空格的文件名时使用 `-0` 选项
2. **参数限制**：系统对命令行长度有限制，使用 `-s` 控制
3. **并行执行**：使用 `-P` 时注意系统资源
4. **错误处理**：某些命令失败不会停止 xargs 继续执行
5. **安全性**：使用 `-p` 在危险操作前确认

### 常见错误和解决方案

#### 1. 参数过长
```bash
# 问题：参数列表太长
find . -name "*.txt" | xargs rm
# 解决：限制参数数量
find . -name "*.txt" | xargs -n 100 rm
```

#### 2. 文件名包含空格
```bash
# 问题：文件名包含空格导致错误
find . -name "*.txt" | xargs rm
# 解决：使用 null 分隔符
find . -name "*.txt" -print0 | xargs -0 rm
```

#### 3. 空输入
```bash
# 问题：输入为空时仍然执行命令
find . -name "*.nonexistent" | xargs rm
# 解决：使用 -r 选项
find . -name "*.nonexistent" | xargs -r rm
```

### 性能优化

1. **并行处理**：使用 `-P` 选项进行并行处理
2. **批量处理**：使用 `-n` 控制每次处理的参数数量
3. **内存使用**：使用 `-s` 控制命令行大小

### 相关命令

- `find`：查找文件
- `parallel`：GNU parallel，更强大的并行处理工具
- `exec`：find 命令的 -exec 选项
- `while read`：shell 循环读取

xargs 是 Unix/Linux 系统中非常实用的命令，特别适合处理大量文件或数据的批量操作。
