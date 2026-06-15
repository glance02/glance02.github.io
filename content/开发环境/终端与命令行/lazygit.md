---
layout:     post
title:      "lazygit 使用方法"
subtitle:   "用终端 TUI 更轻松地操作 Git"
date:       2026-06-15 00:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 笔记
---

`lazygit` 是一个 Git 的终端图形界面工具。它不会替代 Git 本身，而是把 `status`、`add`、`commit`、`push`、`pull`、`branch`、`stash`、`rebase` 等常用操作做成了可视化面板。

适合用在这些场景：

- 想快速查看工作区、暂存区、提交历史和分支。
- 想逐行暂存代码，而不是一次性 `git add .`。
- 想更直观地处理 stash、分支切换、冲突、交互式 rebase。
- 在服务器或 WSL 里不方便打开图形 Git 客户端时。

如果 Git 基础概念还不熟，可以先看 [[useGit|Git 常用流程与命令速查]]。

---

## 1. 安装

### Windows

使用 `winget`：

```powershell
winget install -e --id JesseDuffield.lazygit
```

使用 `scoop`：

```powershell
scoop install lazygit
```

使用 `choco`：

```powershell
choco install lazygit
```

### Linux / WSL

如果发行版仓库里有：

```bash
sudo apt install lazygit
```

也可以用 Go 安装：

```bash
go install github.com/jesseduffield/lazygit@latest
```

如果安装后提示找不到 `lazygit`，检查 Go 的二进制目录是否在 `PATH` 里：

```bash
echo $PATH
```

通常需要把下面这个目录加入 `PATH`：

```bash
~/go/bin
```

---

## 2. 启动

进入任意 Git 仓库目录：

```bash
lazygit
```

常用 alias：

```bash
alias lg='lazygit'
```

可以写进 `~/.bashrc`、`~/.zshrc` 或 PowerShell 配置文件。PowerShell 配置文件可以参考 [[win上的特殊路径]] 里关于 `$PROFILE` 的部分。

---

## 3. 界面理解

lazygit 的界面通常分成几块：

| 面板 | 作用 |
| --- | --- |
| Files | 查看工作区文件，暂存、取消暂存、丢弃修改 |
| Branches | 查看、切换、创建、删除分支 |
| Commits | 查看提交历史，rebase、reset、revert、cherry-pick |
| Stash | 查看和恢复 stash |
| Status / Diff | 查看当前选择对象的详细信息和 diff |

基本操作逻辑：

- 用方向键或 `j` / `k` 上下移动。
- 用 `h` / `l` 或左右方向键在层级间切换。
- 用 `tab` 在主要面板之间切换。
- 用 `?` 查看当前面板可用快捷键。
- 用 `q` 退出。

不确定某个键能做什么时，先按 `?`。这是 lazygit 里最值得记的快捷键。

---

## 4. 日常提交流程

### 4.1 查看改动

启动：

```bash
lg
```

进入后先看 `Files` 面板：

- 左侧文件列表显示修改、新增、删除的文件。
- 右侧会显示当前文件的 diff。
- 选中某个文件后按 `enter`，可以进入更细的 hunk / line 级别视图。

### 4.2 暂存文件

在 `Files` 面板：

| 操作 | 快捷键 |
| --- | --- |
| 暂存 / 取消暂存当前文件 | `space` |
| 暂存 / 取消暂存所有文件 | `a` |
| 进入文件，逐块或逐行暂存 | `enter` |
| 退出当前层级 | `esc` |

逐行暂存很好用：进入文件后，移动到具体行或代码块，按 `space` 只暂存这一部分。

### 4.3 提交

在 `Files` 面板中，确认需要提交的内容已经暂存后：

```text
c
```

输入 commit message，然后回车确认。

如果想用 Git 配置里的编辑器写更长的提交信息：

```text
C
```

### 4.4 推送

```text
P
```

大写 `P` 是 push。第一次推送当前分支时，如果上游分支没有配置，lazygit 会提示设置 upstream。

### 4.5 拉取

```text
p
```

小写 `p` 是 pull。建议在开始工作前先 pull 一下，减少后面冲突的概率。

---

## 5. 分支操作

进入 `Branches` 面板后：

| 操作 | 快捷键 |
| --- | --- |
| 切换到选中分支 | `space` |
| 新建分支 | `n` |
| 按名称检出分支 | `c` |
| 删除分支 | `d` |
| 重命名分支 | `R` |
| 合并选中分支到当前分支 | `M` |
| 将当前分支 rebase 到选中分支 | `r` |
| 设置 / 取消上游分支 | `u` |

理解方式和 Git 命令一致：

- 当前分支是“你所在的位置”。
- merge 是把选中分支合并到当前分支。
- rebase 是把当前分支的提交搬到选中分支后面。

分支操作前最好先看清楚当前分支名，避免把分支合并到错误的位置。

---

## 6. Stash 暂存现场

如果当前工作做了一半，但需要临时切分支，可以用 stash。

在 `Files` 面板：

| 操作 | 快捷键 |
| --- | --- |
| stash 当前改动 | `s` |
| 打开 stash 选项 | `S` |

常见场景：

1. 当前工作没做完。
2. 按 `s` 把改动放进 stash。
3. 切到别的分支处理事情。
4. 回来后在 `Stash` 面板恢复 stash。

恢复 stash 时通常会有 apply / pop 之类选项：

- `apply`：恢复改动，但 stash 记录还保留。
- `pop`：恢复改动，并删除这条 stash。

不确定时先用 `apply`，更稳一点。

---

## 7. 丢弃修改与 reset

lazygit 里有一些操作很方便，也很危险。

在 `Files` 面板：

| 操作 | 快捷键 |
| --- | --- |
| 查看丢弃当前文件修改的选项 | `d` |
| 查看工作区 reset 选项 | `D` |
| 查看上游 reset 选项 | `g` |

注意：

- `d` 通常用于丢弃选中文件的修改。
- `D` 可能涉及清理整个工作区。
- `reset --hard` 类操作会丢掉本地改动，执行前先确认 `Files` 面板里没有要保留的内容。

拿不准时，先退出 lazygit，用命令行看一眼：

```bash
git status
```

---

## 8. 提交历史与 rebase

进入 `Commits` 面板后，可以查看提交历史。

常用操作：

| 操作 | 快捷键 |
| --- | --- |
| 查看提交中的文件 | `enter` |
| 开始交互式 rebase | `i` |
| squash 提交 | `s` |
| fixup 提交 | `f` |
| 修改提交信息 | `r` |
| 删除提交 | `d` |
| 上移提交 | `ctrl+k` |
| 下移提交 | `ctrl+j` |
| 查看 reset 选项 | `g` |
| revert 提交 | `t` |
| 复制提交用于 cherry-pick | `C` |
| 粘贴提交，即 cherry-pick | `V` |

rebase 适合整理自己本地还没共享出去的提交，例如：

- 把多个零散提交 squash 成一个。
- 改掉写得不好的 commit message。
- 调整提交顺序。

已经 push 且别人可能基于它开发的提交，不要随便 rebase。

---

## 9. 冲突处理

发生 merge / rebase 冲突时，lazygit 会在 `Files` 面板显示冲突文件。

处理流程：

1. 选中冲突文件。
2. 按 `enter` 查看冲突块。
3. 按提示选择保留哪一侧，或者按 `e` 打开编辑器手动编辑。
4. 解决完后暂存文件。
5. 按 `m` 打开 merge / rebase 菜单，选择 continue。

对应命令行思路：

```bash
git status
git add <冲突文件>
git rebase --continue
# 或 git merge --continue
```

如果想放弃当前 merge / rebase，也可以按 `m` 打开菜单，选择 abort。

---

## 10. 搜索、过滤和刷新

| 操作 | 快捷键 |
| --- | --- |
| 搜索 / 过滤当前面板 | `/` |
| 刷新 Git 状态 | `R` |
| 执行 shell 命令 | `:` |
| 打开命令日志菜单 | `@` |
| 切换 diff 上下文大小 | `{` / `}` |
| 切换 diff 中是否显示空白字符变化 | `ctrl+w` |

`R` 只是刷新 lazygit 看到的 Git 状态，不等于 `git fetch`。如果要从远程抓取更新，在 `Files` 面板可以按 `f`。

---

## 11. 最小必记快捷键

| 快捷键 | 作用 |
| --- | --- |
| `?` | 查看当前面板帮助 |
| `q` | 退出 |
| `tab` | 切换面板 |
| `j` / `k` | 上下移动 |
| `h` / `l` | 左右切换层级 |
| `space` | 暂存 / 取消暂存，或切换选中状态 |
| `enter` | 进入当前项目 |
| `esc` | 返回上一层 / 取消 |
| `c` | 提交 |
| `p` | pull |
| `P` | push |
| `s` | stash |
| `/` | 搜索 |
| `R` | 刷新 |

---

## 12. 我的使用习惯

日常改代码：

```text
lg -> 看 Files -> space 暂存 -> c 提交 -> P 推送
```

只提交部分代码：

```text
lg -> 选文件 -> enter -> 选 hunk/line -> space -> esc -> c
```

临时切分支：

```text
lg -> s stash -> Branches 面板切分支 -> 回来后恢复 stash
```

整理提交：

```text
lg -> Commits 面板 -> i 交互式 rebase -> s/f/r 整理提交
```

最重要的原则：**lazygit 是 Git 的可视化入口，但判断操作风险时仍然按 Git 的规则来**。遇到 `reset`、`drop`、`force checkout`、`rebase` 这类会改历史或丢改动的操作，先确认当前分支和工作区状态。

---

## 相关笔记

- [[useGit|Git 常用流程与命令速查]]
- [[ssh]]
- [[tmux 基本用法总结]]
- [[win上的特殊路径]]
