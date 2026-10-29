---
title: Git Commands
tags:
  - Git
categories:
  - Tool
date: 2026-06-24 00:00:00
katex: true
---

# 1. 仓库初始化与远程

## 1.1. git init：在当前目录建立仓库

生成 `.git/` 之后，该目录及子目录的改动都可被 Git 记录。在已有仓库上重复执行不会清空历史，只补齐缺失的元数据。

```bash
git init
```

## 1.2. git remote add 与 git remote -v：登记并核对远程地址

`git remote add <name> <url>` 把远程地址登记为别名 `<name>`，惯例用 `origin`，同一远程在不同仓库里可以起不同名字，别名已存在时命令报错拒绝覆盖。`-v` 列出每个别名的 fetch 与 push 地址，推送前用它确认地址没有写错。

```bash
git remote add origin git@example.com:user/repo.git
git remote -v
```

## 1.3. git push -u：首推并建立跟踪

`-u` 把本地 `main` 与 `origin/main` 绑定为跟踪关系，之后 `git push`、`git pull` 可以省略远程名与分支名，只有首次推送需要。命令里的分支名必须与本地实际分支一致，旧仓库默认分支常为 `master`，先用 `git branch` 确认。

```bash
git push -u origin main
```

# 2. 查看状态与差异

## 2.1. git status：确认工作区与暂存区状态

输出当前分支与跟踪关系、已暂存待提交的改动、未暂存与未跟踪的文件。提交前用它检查是否漏加或多加文件；该命令只读状态，不修改任何内容。

```bash
git status
```

## 2.2. git diff 与 git diff --staged：分别查看未暂存与已暂存的修改

`git diff` 比较工作区与索引（index，即暂存区的正式名称），只显示改过但还没 `git add` 的内容；`git diff --staged` 比较索引与 `HEAD`，显示下一次提交会写进历史的内容。`--staged` 与 `--cached` 等价。某处改动暂存后又改了工作区，新增部分只出现在 `git diff` 里。

```bash
git diff            # 未暂存的改动
git diff --staged   # 即将提交的内容
```

# 3. 暂存与文件操作

## 3.1. git add：把改动放进暂存区

`git add <path>...` 暂存列出的文件，`git add .` 暂存当前目录下的修改、新增与删除。只有进入暂存区的内容才会被提交；该命令不产生提交，重复执行也安全。

```bash
git add src/app.js src/util.js
```

## 3.2. git mv：移动或重命名并暂存

把文件从 `<old>` 移动或重命名到 `<new>` 并暂存改动，等价于移动文件后再 `git add`；Git 按内容比对识别重命名，所以用系统命令移动再暂存效果相同。源文件必须已被跟踪，否则命令报错拒绝执行。

```bash
git mv old-name.md new-name.md
```

## 3.3. git rm：删除文件并暂存删除

删除工作区文件并把删除写入索引，提交后文件从最新版本中消失，早先版本仍可查看。文件存在未提交修改时命令拒绝执行以免误删，删除目录需要加 `-r`。

```bash
git rm -r old-dir/
```

## 3.4. git rm --cached：只从索引移除，保留工作区文件

只让 Git 不再跟踪该路径，磁盘上的文件原样保留，适合把误提交的构建产物移出版本控制，目录需要加 `-r`。执行后该文件变成未跟踪，此后 `.gitignore` 才对它生效；索引变化必须提交一次才真正落到历史里。

```bash
git rm -r --cached dist/
```

# 4. 提交

## 4.1. git commit -m：提交暂存区内容

把暂存区的当前内容写成一次提交，工作区中未暂存的改动留在原处；暂存区为空时提交失败，需要先 `git add`。要写多段信息时重复一次 `-m`，或改用编辑器。

```bash
git commit -m "fix(parser): 修正空输入时的越界读取"
```

## 4.2. git commit -v：提交前在编辑器里核对 diff

打开提交信息编辑器时在下方插入本次提交的 diff，便于逐行确认改动；这段 diff 是注释，不会写进提交信息。该选项只在编辑器被打开时生效，与 `-m` 同用时看不到 diff。

```bash
git commit -v
```

## 4.3. git commit <file>：只提交指定路径

这种形式绕过索引：`git commit <path>... -m "<message>"` 只提交命令行列出的路径，内容取自这些文件在工作区中的当前版本，这些路径已经暂存的内容被忽略；未列出的暂存改动仍留在索引里，不进入本次提交。新增文件必须先 `git add`，否则报错 `pathspec ... did not match`。日常更推荐先显式暂存再统一提交。

```bash
git commit config.json -m "chore: 更新配置"
```

# 5. 查看历史

## 5.1. git log 与 git show：列出历史与查看单个提交

`git log` 按时间倒序列出提交哈希、作者、日期与信息，`--oneline --graph --all` 组合成紧凑的分支图。`git show` 显示指定提交的改动，参数可以是哈希、分支名或引用表达式，不带参数时默认查看 `HEAD`。

```bash
git log --oneline --graph --all
git show HEAD
```

## 5.2. HEAD、^ 与 ~：定位祖先提交的引用语法

`HEAD` 指当前检出的提交。`^` 取父提交：`HEAD^` 是第一个父提交，`HEAD^5` 是第 5 个父提交，只有合并提交才可能有多于一个父提交，在普通提交上执行 `HEAD^5` 会因引用不存在而报错。`~` 沿第一父提交链向上数代：`HEAD~5` 是往前第 5 个祖先提交，与“往前 5 个版本”的直觉一致。两者的差别只在合并提交上体现。

```bash
git show HEAD~5   # 往前第 5 个祖先提交
git show HEAD^5   # 第 5 个父提交，通常只用于合并提交
```

# 6. 分支

## 6.1. git branch 与 git branch -m：列出、创建、重命名分支

`git branch` 不带参数列出本地分支，当前分支前有 `*`；`git branch <name>` 创建分支但不切换。`git branch -m <old> <new>` 只移动本地分支引用，不改动远程分支，省略 `<old>` 时重命名当前分支，目标名字已存在则报错。

```bash
git branch
git branch -m old-name new-name
```

## 6.2. git switch 与 git switch -c：切换与创建并切换

切换后 `HEAD` 指向该分支，工作区随之更新；工作区存在未提交改动且与新分支冲突时切换被拒绝，需要先提交这些改动。`-c` 新建分支并立即切换，起点是当前 `HEAD`。`switch` 与 `restore` 从 Git 2.23 起提供，更早的版本只有 `checkout`。

```bash
git switch main
git switch -c fix-login
```

## 6.3. git checkout -：回到上一个分支

在最近使用的两个分支之间来回切换，用于快速回到刚离开的分支；它只切换分支，不撤销任何文件改动。

```bash
git checkout -
```

## 6.4. git merge：把另一分支合并进当前分支

先切到接收方分支再执行，指定分支的提交由此进入当前分支。两侧改到同一处会产生冲突，冲突文件保留标记，手工解决后 `git add` 并提交完成合并。接收方分支自两侧分叉以来没有新提交时可以快进（fast-forward），只把分支指针前移，不产生新提交；否则产生一个合并提交。

```bash
git switch main
git merge feature-login
```

# 7. 与远程同步

## 7.1. git fetch：只取回远程更新

下载远程的新提交并更新 `origin/*` 这类远程跟踪引用，本地分支与工作区都不动，因此随时执行都安全。要判断远程多了什么内容，取回后查看 `origin/main` 即可。

```bash
git fetch origin
git log origin/main   # 远程分支当前指向的提交
```

## 7.2. git pull 与 git pull --rebase：取回后合并或变基

`git pull` 等价于 `git fetch` 加 `git merge origin/<当前分支>`，既更新远程跟踪引用，也改动本地分支，本地有未推送提交时会形成合并提交或产生冲突。`--rebase` 把本地未推送的提交重放到远程提交之后，历史保持一条直线；重放会重写这些提交并改变哈希，已经被推送、别人可能引用的提交不要这样处理，改用 `git merge` 或 `git revert`。

```bash
git pull --rebase
```

## 7.3. git push：把本地提交送到远程

只推送已提交的内容，未提交的改动留在本地；远程分支含有本地没有的提交时推送被拒绝，需要先取回并合并。首次推送用 `git push -u origin main` 建立跟踪关系。

```bash
git push
```

# 8. 撤销与回退

## 8.1. git restore 与 git restore --staged：丢弃工作区修改或取消暂存

`git restore <path>` 用索引中的版本覆盖工作区文件，**未暂存的修改无法找回**。该命令只处理已跟踪文件，不能靠它删除新文件。`--staged` 把路径在索引中的内容还原到 `HEAD` 的版本，工作区文件内容不变，用于撤回加错的暂存，改动回到未暂存状态。

```bash
git restore app.js
git restore --staged app.js
```

## 8.2. git reset --hard：分支、索引、工作区一起回退

`git reset --hard <commit>` 把分支引用、索引与工作区一起回退：分支引用指向目标提交，索引与工作区都改成该提交的内容。**暂存区与工作区里未提交的修改全部丢失**，目标提交之后的本地提交也从分支上消失，只有未跟踪文件不受影响，因此只适合尚未推送、无人依赖的本地历史。

```bash
git reset --hard HEAD~1   # 丢弃最近一次提交及其全部改动
```

## 8.3. git revert：用反向提交撤销

将上一个提交作为新提交而提交，历史只增不改，因此适合已经推送、别人可能已经基于它工作的提交，对方正常拉取即可获得撤销效果。撤销合并提交需要额外指明保留哪一侧的父提交。

```bash
git revert HEAD
```

# 9. Conventional Commits 提交规范

## 9.1. 提交信息格式：一行标题加可选正文与脚注

标题行写成 `<type>(<scope>): <subject>`，之后空一行写 body，再空一行写 footer；scope 可省略，写成 `fix: 修正拼写` 同样合法。

```text
feat(auth): 添加用户登录功能

使用 JWT 实现登录状态保持，在 token 过期时要求重新登录。

closes #123
```

## 9.2. 字段说明与 type 表：各部分的必填性与取值

| 部分 | 说明 |
|---|---|
| `type` | 提交类型，必填，取值见下表 |
| `scope` | 影响范围或模块，可选，用括号包裹 |
| `subject` | 一行简短描述，必填 |
| `body` | 详细说明，可选，写改动动机与影响 |
| `footer` | issue 关联、`BREAKING CHANGE` 等，可选 |

| type | 含义 |
|---|---|
| `feat` | 新功能 |
| `fix` | 修复 bug |
| `docs` | 文档改动 |
| `style` | 格式、缩进、空格等不影响逻辑的修改 |
| `refactor` | 重构，既不新增功能也不修 bug |
| `test` | 添加或修改测试 |
| `chore` | 构建过程、依赖、辅助工具等杂项 |
| `perf` | 性能优化 |
| `ci` | CI 配置相关修改 |
| `revert` | 回滚提交 |

## 9.3. 写作要求：标题一句话说清改动

subject 短到一眼看懂，英文项目用祈使句 `add login page`，不写 `added login page`；中文项目统一用动宾短语，如 `完善 SSH 端口转发说明`。body 说明为什么改与影响是什么，不复述 diff。修复 issue 时在 footer 写 `closes #123`，破坏兼容时写以 `BREAKING CHANGE:` 起头的说明。

```text
refactor(cache): 用 LRU 替换固定容量淘汰

旧实现按写入顺序淘汰，热点数据在批量写入后频繁失效。

BREAKING CHANGE: 缓存容量配置项由 size 改为 capacity。
```

## 9.4. 提交模板与 commitizen：把规范固化到流程里

模板文件内容会在打开提交编辑器时作为初始文本出现，被注释的行不进入提交信息；该配置只在编辑器被打开时生效，用 `-m` 直接给出信息时不读取模板，路径需指向真实存在的文件。commitizen 按提示依次询问 type、scope 与描述并格式化结果，提问内容由配置的适配器决定，`cz-conventional-changelog` 对应本节格式。

```text
<type>(<scope>): <subject>

# 改动动机：
# 主要实现：
# Footer:
# closes #issue
# BREAKING CHANGE: xxx
```

```bash
git config --global commit.template ~/.gitmessage.txt
npm install -g commitizen   # -g 全局安装，可能需要管理员权限
cz
```

# 10. 语义化版本

版本号 `MAJOR.MINOR.PATCH` 的三段数字分别对应不兼容改动、向后兼容的新功能、向后兼容的修复，递增高位时低位归零，例如 `1.4.2` 加入新功能后是 `1.5.0`，出现不兼容修改后是 `2.0.0`。`0.y.z` 阶段的 API 视为不稳定。提交前缀只是判断线索，版本号仍需按真实改动判定。

| 版本位 | 递增时机 | 对应提交前缀 |
|---|---|---|
| `MAJOR` | 不兼容 API 修改 | `BREAKING CHANGE` |
| `MINOR` | 向后兼容的新功能 | `feat` |
| `PATCH` | 向后兼容的问题修复 | `fix` |

# 11. 文件忽略

## 11.1. .gitignore 语法：每行一条匹配规则

路径相对 `.gitignore` 所在目录，`/` 结尾表示目录，`*` 可以出现在路径中间与结尾，`!` 开头把此前被忽略的路径重新纳入。被排除的目录整体不再向下遍历，其内部文件无法单独用 `!` 取回，需要先让目录本身不被排除。

```gitignore
*.log
/dist/
!logs/example.log
```

## 11.2. 只对未跟踪文件生效

`.gitignore` 只影响未被跟踪的文件。已经提交过的文件之后再加进 `.gitignore` 不会停止跟踪，改动后照样出现在 `git status` 中，显示为 `M` 或 `D`；要让 Git 以后不再跟踪它，必须先用 `git rm --cached` 从索引移除并提交这次移除。

```bash
git rm -r --cached <path>
```
