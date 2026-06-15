---
layout:     post
title:      "Git 常用流程与命令速查"
subtitle:   "给自己用的 Git 工作流备忘"
date:       2026-03-15 14:06:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 笔记
---

Git 的核心不是背命令，而是理解代码在几个位置之间流动：

```
工作区 -> 暂存区 -> 本地仓库 -> 远程仓库
```

- **工作区**：当前文件夹里正在编辑的文件。
- **暂存区**：用 `git add` 选中的、准备提交的改动。
- **本地仓库**：用 `git commit` 保存下来的历史版本。
- **远程仓库**：GitHub、GitLab、服务器上的仓库，用 `push` / `pull` 同步。

远程仓库通常需要配合 [[ssh]] 密钥使用；如果是在服务器上编译源码，也会和 [[n2n Supernode 部署]] 这类笔记中的 `git clone` 流程关联。

---

## 1. 最常用工作流

### 1.1 第一次创建仓库

本地已有项目，想推到 GitHub：

```bash
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin <远程仓库地址>
git push -u origin main
```

说明：

- `git init`：把当前目录初始化成 Git 仓库。
- `git branch -M main`：把当前分支改名为 `main`。
- `git remote add origin ...`：把远程仓库命名为 `origin`。
- `git push -u origin main`：第一次推送，并建立本地 `main` 与远程 `origin/main` 的追踪关系。

### 1.2 从远程仓库拉代码

```bash
git clone <远程仓库地址>
```

如果只想更新当前仓库：

```bash
git pull
```

`git pull` 大致等价于：

```bash
git fetch
git merge
```

也就是说，它会先从远程拉取更新，再尝试合并到当前分支。

### 1.3 日常提交

```bash
git status
git add <文件名>
git commit -m "描述这次修改"
git push
```

常用习惯：

- 提交前先 `git status`，确认自己改了什么。
- 尽量用 `git add <文件名>` 精确添加，避免把临时文件一起提交。
- commit message 写“做了什么”，不要只写 `update`。

如果确定所有改动都要提交：

```bash
git add .
git commit -m "描述这次修改"
```

---

## 2. 查看状态与历史

### 2.1 查看当前状态

```bash
git status
```

它会告诉你：

- 哪些文件被修改了。
- 哪些文件已经进入暂存区。
- 当前在哪个分支。
- 当前分支和远程分支是否有差异。

### 2.2 查看提交历史

```bash
git log --oneline --graph --decorate --all
```

这个命令适合看分支关系：

- `--oneline`：每个提交只显示一行。
- `--graph`：用图形显示分支结构。
- `--decorate`：显示分支名、tag 等引用。
- `--all`：显示所有分支。

### 2.3 查看具体改动

查看工作区中还没暂存的改动：

```bash
git diff
```

查看已经暂存、准备提交的改动：

```bash
git diff --cached
```

查看某次提交改了什么：

```bash
git show <commit-id>
```

---

## 3. 分支管理

### 3.1 查看分支

```bash
git branch
git branch -a
git branch -r
```

- `git branch`：查看本地分支。
- `git branch -a`：查看本地和远程分支。
- `git branch -r`：只查看远程分支。

### 3.2 创建和切换分支

推荐使用 `switch`，语义更清楚：

```bash
git switch main
git switch -c develop
```

旧写法也可以：

```bash
git checkout main
git checkout -b develop
```

含义：

- `git switch main`：切换到 `main` 分支。
- `git switch -c develop`：创建 `develop` 分支，并切换过去。

### 3.3 合并分支

把 `develop` 合并进 `main`：

```bash
git switch main
git merge develop
```

理解方式：**当前在哪个分支，合并结果就会进入哪个分支**。

如果合并时出现冲突：

```bash
git status
# 手动打开冲突文件，保留正确内容
git add <冲突文件>
git commit
```

### 3.4 删除分支

删除本地分支：

```bash
git branch -d <branch-name>
```

如果这个分支还没被合并，Git 会阻止删除。确认不要这个分支时再强制删除：

```bash
git branch -D <branch-name>
```

删除远程分支：

```bash
git push origin --delete <branch-name>
```

---

## 4. 远程仓库

### 4.1 查看远程仓库

```bash
git remote -v
```

### 4.2 添加远程仓库

```bash
git remote add origin <远程仓库地址>
```

### 4.3 修改远程仓库地址

```bash
git remote set-url origin <新的远程仓库地址>
```

### 4.4 推送当前分支

第一次推送：

```bash
git push -u origin <branch-name>
```

之后可以简写：

```bash
git push
```

### 4.5 拉取远程更新

```bash
git fetch origin
```

`fetch` 只更新远程引用，不会直接改工作区。想先看看远程发生了什么，用它更稳。

---

## 5. 撤销与回退

撤销类命令最容易误伤，先分清楚你要撤销的是哪一层。

### 5.1 撤销工作区修改

丢弃某个文件尚未暂存的修改：

```bash
git restore <文件名>
```

### 5.2 取消暂存

文件已经 `git add`，但还没 commit：

```bash
git restore --staged <文件名>
```

### 5.3 修改上一次提交

刚 commit 完，发现漏了文件或提交信息写错：

```bash
git add <漏掉的文件>
git commit --amend
```

如果这个提交已经推到远程，并且别人可能基于它继续开发，就不要随便 amend。

### 5.4 回到历史版本

危险操作：

```bash
git reset --hard <commit-id>
```

它会让当前分支回到指定提交，并丢弃之后的工作区改动。使用前至少确认：

```bash
git status
git log --oneline
```

如果只是想“做一个反向提交”来撤销历史提交，更适合用：

```bash
git revert <commit-id>
```

`revert` 会生成一个新提交，不会改写历史，更适合已经推送到远程的分支。

### 5.5 找回误删的提交

```bash
git reflog
```

`reflog` 会记录 HEAD 移动历史。即使某些提交不在 `git log` 里了，也可能还能从这里找到。

---

## 6. rebase

`rebase` 的作用是把一串提交“搬到”另一个提交后面，让历史看起来更线性。

### 6.1 用 main 更新当前分支

在 feature 分支上，把 main 的最新提交接到自己前面：

```bash
git switch feature
git fetch origin
git rebase origin/main
```

理解方式：

```
先取 main 的最新位置，再把 feature 上自己的提交重新放到 main 后面
```

### 6.2 rebase 冲突处理

如果出现冲突：

```bash
git status
# 手动解决冲突
git add <冲突文件>
git rebase --continue
```

放弃这次 rebase：

```bash
git rebase --abort
```

跳过当前这个提交：

```bash
git rebase --skip
```

### 6.3 交互式 rebase

整理最近 3 个提交：

```bash
git rebase -i HEAD~3
```

常用操作：

| 指令 | 作用 |
| --- | --- |
| `pick` | 保留提交 |
| `reword` | 修改提交信息 |
| `edit` | 停在该提交，允许修改内容 |
| `squash` | 合并到前一个提交，并保留提交信息 |
| `fixup` | 合并到前一个提交，丢弃当前提交信息 |
| `drop` | 删除提交 |

注意：已经推送到远程、并且别人可能使用的提交，不要轻易 rebase。

---

## 7. .gitignore

`.gitignore` 用来声明哪些文件不需要 Git 追踪，例如：

```gitignore
node_modules/
dist/
*.log
*.exe
.env
```

注意：`.gitignore` 只对**还没有被 Git 追踪的文件**生效。已经提交过的文件，即使后来写进 `.gitignore`，Git 仍然会继续追踪。

### 7.1 让已追踪文件停止被追踪

先从 Git 缓存中移除，不删除本地文件：

```bash
git rm --cached <文件名>
```

如果是目录：

```bash
git rm -r --cached <目录名>
```

然后提交：

```bash
git add .gitignore
git commit -m "update gitignore"
```

查看当前被 Git 追踪的文件：

```bash
git ls-files
```

---

## 8. 常见问题

### 8.1 本地 main 和远程 main 没有关联

第一次推送时使用：

```bash
git push -u origin main
```

以后就可以直接：

```bash
git push
git pull
```

### 8.2 远程默认分支是 master 或 main

查看远程分支：

```bash
git branch -r
```

如果本地分支名需要改成 `main`：

```bash
git branch -M main
```

### 8.3 本地仓库想接入一个已有远程仓库

```bash
git remote add origin <远程仓库地址>
git fetch origin
git branch -a
```

如果本地历史和远程历史本来就是同一个项目，通常可以：

```bash
git pull origin main
```

如果两边是完全无关的历史，Git 会拒绝合并。确认确实要把两个无关项目合在一起时，再用：

```bash
git pull origin main --allow-unrelated-histories
```

这不是日常命令，只适合“两个仓库历史本来不相关，但现在必须合并”的场景。

### 8.4 强制让本地变成远程某个分支

危险操作，适合确认本地改动都不要了的情况：

```bash
git fetch origin
git reset --hard origin/main
```

这会把本地当前分支直接重置成远程 `main` 的状态。执行前先看：

```bash
git status
```

---

## 9. 个人速查

| 场景 | 命令 |
| --- | --- |
| 查看状态 | `git status` |
| 查看历史 | `git log --oneline --graph --decorate --all` |
| 查看未暂存改动 | `git diff` |
| 查看已暂存改动 | `git diff --cached` |
| 添加文件 | `git add <文件名>` |
| 提交 | `git commit -m "message"` |
| 推送 | `git push` |
| 拉取并合并 | `git pull` |
| 只拉取远程信息 | `git fetch origin` |
| 查看分支 | `git branch -a` |
| 切换分支 | `git switch <branch>` |
| 新建并切换分支 | `git switch -c <branch>` |
| 合并分支 | `git merge <branch>` |
| 取消暂存 | `git restore --staged <文件名>` |
| 丢弃工作区改动 | `git restore <文件名>` |
| 查看 HEAD 历史 | `git reflog` |

---

## 相关笔记

- [[ssh]]
- [[vimLearn]]
- [[tmux 基本用法总结]]
