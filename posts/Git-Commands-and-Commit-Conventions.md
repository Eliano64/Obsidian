---
title: Git Commands and Commit Conventions
tags:
  - Git
categories:
  - Tool
date: 2026-06-24 00:00:00
katex: true
---


# 1. 初始化仓库与关联远程仓库

```bash
git init
```

在当前目录初始化一个 Git 仓库。

```bash
git remote add origin <url>
git remote -v
```

- `git remote add origin <url>`：把远程仓库地址添加为 `origin`。
- `git remote -v`：查看当前关联的远程仓库。

首次推送主分支：

```bash
git push -u origin main
```

`-u` 会设置本地 `main` 与远程 `origin/main` 的跟踪关系，之后可以直接使用 `git push` / `git pull`。

# 2. 查看状态与差异

```bash
git status
git diff
git diff --staged
```

| 命令 | 作用 |
|---|---|
| `git status` | 查看工作区、暂存区状态 |
| `git diff` | 查看工作区尚未暂存的修改 |
| `git diff --staged` | 查看已经暂存、即将提交的修改 |

# 3. 添加、移动、删除文件

```bash
git add <file1> <file2>
git add .
git mv <old> <new>
git rm <file>
```

| 命令 | 作用 |
|---|---|
| `git add <file>` | 把指定文件加入暂存区 |
| `git add .` | 暂存当前目录下的修改、新增、删除 |
| `git mv <old> <new>` | 重命名/移动文件，并把操作加入暂存区 |
| `git rm <file>` | 删除文件，并把删除加入暂存区 |

如果文件已经被 Git 跟踪，后来才加入 `.gitignore`，不会自动停止跟踪。需要先从索引中移除：

```bash
git rm -r --cached <path>
```

# 4. 提交代码

```bash
git commit -m "<message>"
git commit -v
```

| 命令 | 作用 |
|---|---|
| `git commit -m "msg"` | 提交暂存区内容，并写一行提交信息 |
| `git commit -v` | 提交时在编辑器中显示 diff，便于确认改动 |
| `git commit <file> -m "msg"` | 提交指定文件中已经暂存或可提交的改动；日常更推荐先 `git add` 再统一提交 |

# 5. 查看历史与提交详情

```bash
git log
git log --oneline --graph --all
git show <commit_id>
git show HEAD
git show HEAD^
```

- `HEAD`：当前检出的提交。
- `HEAD^`：当前提交的父提交。
- `HEAD~5`：向前第 5 个祖先提交；比 `^5` 更符合“往前 5 个版本”的直觉。`HEAD^5` 表示第 5 个父提交，通常只在合并提交中有意义。

# 6. 分支管理

```bash
git branch
git branch <branch_name>
git branch -m <old> <new>
git switch <branch_name>
git switch -c <branch_name>
git checkout -
git merge <branch_name>
```

| 命令 | 作用 |
|---|---|
| `git branch` | 列出本地分支 |
| `git branch <name>` | 创建分支但不切换 |
| `git branch -m <old> <new>` | 重命名本地分支 |
| `git switch <name>` | 切换分支 |
| `git switch -c <name>` | 创建并切换分支 |
| `git checkout -` | 回到上一个分支 |
| `git merge <name>` | 把指定分支合并到当前分支 |

`checkout` 仍然可用，但新版本 Git 更推荐用 `switch` 处理分支切换，用 `restore` 处理文件恢复，语义更清楚。

# 7. 拉取、获取与推送

```bash
git fetch origin
git pull
git pull --rebase
git push
```

| 命令 | 作用 |
|---|---|
| `git fetch` | 获取远程更新，但不自动合并；相对安全 |
| `git pull` | `fetch + merge`，拉取并合并到当前分支 |
| `git pull --rebase` | 拉取后把本地提交 rebase 到远程提交之后 |
| `git push` | 推送本地提交到远程 |

# 8. 回退与撤销

```bash
git restore <file>
git restore --staged <file>
git reset --hard <commit_id>
git revert <commit_id>
```

| 命令 | 作用 | 风险 |
|---|---|---|
| `git restore <file>` | 丢弃工作区修改 | 会丢失未提交内容 |
| `git restore --staged <file>` | 取消暂存，不改工作区 | 低 |
| `git reset --hard <commit_id>` | 强制回到某个提交，工作区也回退 | 高，可能丢失提交后的修改 |
| `git revert <commit_id>` | 新增一个“反向提交”来撤销某次提交 | 适合公共分支 |

经验规则：

- 自己本地还没推送的历史，可以谨慎 `reset`。
- 已经推送、别人可能基于它工作的历史，优先 `revert`。

# 9. Conventional Commits 提交规范

推荐格式：

```text
<type>(<scope>): <subject>

<optional body>

<optional footer>
```

示例：

```text
feat(auth): 添加用户登录功能

使用 JWT 实现用户登录状态保持，并引入 token 过期检查。
```

## 字段说明

| 部分 | 说明 |
|---|---|
| `type` | 提交类型，必填 |
| `scope` | 影响范围/模块，可选 |
| `subject` | 一行简短描述，必填 |
| `body` | 详细说明，可选，适合说明动机和影响 |
| `footer` | issue 关联、BREAKING CHANGE 等，可选 |

## 常见 type

| type | 含义 |
|---|---|
| `feat` | 新功能 |
| `fix` | 修复 bug |
| `docs` | 文档改动 |
| `style` | 格式、缩进、空格等不影响逻辑的修改 |
| `refactor` | 重构，不新增功能也不修 bug |
| `test` | 添加或修改测试 |
| `chore` | 构建过程、依赖、辅助工具等杂项 |
| `perf` | 性能优化 |
| `ci` | CI 配置相关修改 |
| `revert` | 回滚提交 |

## 提交信息写作建议

- subject 简短明确，尽量一眼看懂改动。
- 英文项目常用祈使句：`add login page`，不要写成 `added login page`。
- 中文项目可以统一用动宾短语：`完善 SSH 端口转发说明`。
- `body` 写“为什么改”和“影响是什么”，不要只重复代码 diff。
- 修复 issue 时可在 footer 写：`closes #123`。
- 破坏兼容时写：`BREAKING CHANGE: ...`。

## commit 辅助工具

### commitizen

```bash
npm install -g commitizen
cz
```

它会引导输入 type、scope、message，并自动格式化提交信息。

### git commit 模板

模板文件示例：

```text
<type>(<scope>): <subject>

# 改动动机：
# 主要实现：
# 额外说明：

# Footer:
# closes #issue
# BREAKING CHANGE: xxx
```

配置：

```bash
git config --global commit.template ~/.gitmessage.txt
```

# 10. 语义化版本 SemVer

版本号格式：

```text
MAJOR.MINOR.PATCH
```

| 部分 | 递增时机 |
|---|---|
| `MAJOR` | 不兼容 API 修改 |
| `MINOR` | 向后兼容的新功能 |
| `PATCH` | 向后兼容的问题修复 |

提交规范和语义化版本可以配合使用：例如 `feat` 对应 minor，`fix` 对应 patch，`BREAKING CHANGE` 对应 major。

# 11. 文件忽略

`.gitignore` 用于声明不应纳入版本控制的文件和目录。每行是一条匹配规则，路径相对于 `.gitignore` 所在目录；以 `/` 结尾表示目录，以 `!` 开头可重新纳入此前被忽略的路径。例如：

```gitignore
# 忽略所有日志文件
*.log

# 忽略项目根目录下的构建产物
/dist/

# 保留日志目录中的示例文件
!logs/example.log
```

但`.gitignore` 只影响“未被 Git 跟踪”的文件。若文件以前已经被提交进仓库了，那么即使之后加 ignore，修改后它们仍会显示为 `M` / `D`。要让 Git 以后不再跟踪它们，需要从索引移除：

```
git rm -r --cached <name>
```
