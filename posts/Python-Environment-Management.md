---
title: Python Environment Management
tags:
  - Python
categories:
  - Tool
date: 2026-09-15 18:20:00
katex: true
---
# `pip install`

`pip install` 默认装进当前解释器所在的 site-packages。系统解释器或 Anaconda 的 base 环境被所有项目共用，误装一次就会影响其它项目，所以先确认当前解释器是哪一个：

```bash
python -c "import sys; print(sys.executable)"
```

# `.venv`

```bash
python -m venv .venv      # 在项目根目录建立 .venv
```

激活命令随平台不同，Windows 下 venv 会同时生成三个激活脚本：

| 平台 | 激活命令 |
|---|---|
| Windows PowerShell | `.\.venv\Scripts\Activate.ps1` |
| Windows cmd | `.venv\Scripts\activate.bat` |
| Git Bash、Linux、macOS | `source .venv/bin/activate` |

退出环境用 `deactivate`。

激活后应当确认解释器确实换了：

```bash
python -c "import sys; print(sys.prefix)"   # 输出应指向 .venv 目录
```

`.venv` 属于本地生成物，写进 `.gitignore`，不要提交。

装包统一用 `python -m pip` 而不是 `pip`：前者明确使用当前解释器的 pip，不会因为 PATH 中的 `pip` 指向另一个环境而装错地方。

# 依赖清单的导出与还原

`pip freeze` 的输出不适合直接当作 requirements.txt。在 conda 环境里执行 `pip freeze`，记录的是打包时的构建目录，换一台机器无法访问，这样的文件装不上。原因有两点：`pip freeze` 输出的是当前环境已安装的全部包，包括作为传递依赖装进来的那些；解释器来自 conda 时，很多包由 conda 构建，pip 只能看到这样的构建路径。

更稳妥的做法：

- 需要当前环境的完整快照时用 `pip list --format=freeze`。注意在全新的 venv 里 `pip freeze` 是空输出，而 `pip list --format=freeze` 会一并列出 pip、setuptools 这类基础包。
- 还原命令是 `python -m pip install -r requirements.txt`。
- 需要防止误装进全局环境时，用 `python -m pip install --require-virtualenv ...`，它会在不在虚拟环境时直接报错退出。

版本约束写法：`==` 精确固定，`~=` 允许补丁位浮动。要复现同样的构建结果时用 `==`。

# conda 环境与导出

conda 除解释器之外还管理带二进制依赖的包，例如 numpy 依赖的 BLAS，这类包用 pip 安装通常要自行编译。

```bash
conda create -n demo python=3.12   # 新建环境并指定解释器版本
conda activate demo                # 切换进该环境
conda env list                     # 列出所有环境
conda env remove -n demo           # 删除整个环境
```

导出与迁移：

```bash
conda env export > environment.yml                 # 完整环境，含平台相关 build 号
conda env export --from-history > environment.yml  # 只含显式安装过的包
conda env export --no-builds > environment.yml     # 去掉 build 号，提高跨平台可用性
conda env create -f environment.yml                # 按文件重建环境
conda env update -f environment.yml --prune        # 更新已有环境，并删除清单之外的包
```

跨机器迁移用 `--from-history`。完整导出会写入平台相关的 build 号，换一个平台或换一个 conda 版本常常解不出依赖；`--from-history` 依据的是安装记录，只保留手动装过的包。若环境里还有 pip 装的包，导出文件的 `pip:` 段会一并记录，重建时由 pip 安装。

conda 与 pip 混用时要注意顺序：先用 conda 装它能提供的包，再用 pip 装剩下的。两者写的是同一份 site-packages，顺序颠倒时 pip 装好的版本容易被后续的 conda 操作覆盖。