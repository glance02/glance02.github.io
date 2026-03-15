---
layout:     post
title:      "tmux 基本用法总结"
subtitle:   "终端多路复用工具 tmux 的常用命令速查"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 项目笔记
---
## 1️⃣ 启动 tmux 会话

新建一个会话（session）：
```bash
tmux new -s session_name
```

例子：
```bash
tmux new -s train
```
- `train` 是会话名字
- 会打开一个新的终端窗口，你可以直接运行训练脚本

---

## 2️⃣ 前缀键

**所有 tmux 命令都需要先按前缀键：**
```
Ctrl + B
```
意思：告诉 tmux **接下来输入的不是普通字符，而是 tmux 命令**。

---

## 3️⃣ 常用操作

| 操作              | 快捷键                            | 说明                         |
| --------------- | ------------------------------ | -------------------------- |
| 离开 tmux（保持程序运行） | Ctrl+B D                       | detach，会话继续在后台运行           |
| 创建新窗口           | Ctrl+B C                       | 相当于打开新的标签页                 |
| 切换窗口            | Ctrl+B N / P 或 Ctrl+B 0-9      | N 下一个窗口，P 上一个窗口，0-9直接切窗口   |
| 垂直分屏            | Ctrl+B %                       | 左右分屏                       |
| 水平分屏            | Ctrl+B "                       | 上下分屏                       |
| 切换 pane（分屏）     | Ctrl+B 方向键                     | 在多个 pane 之间切换              |
| 放大当前 pane       | Ctrl+B Z                       | 当前 pane 全屏，再按一次恢复          |
| 关闭当前 pane       | Ctrl+B X                       | 关闭当前分屏窗口                   |
| 进入复制模式（查看历史输出）  | Ctrl+B [                       | 使用方向键滚动历史输出，Q 退出           |
| detach 并强制接管会话  | tmux attach -d -t session_name | 如果会话被其他客户端占用，可以用 `-d` 强制接管 |

---

## 4️⃣ 查看会话和窗口

- 查看所有会话：
```bash
tmux ls
```

- 重新连接某个会话：
```bash
tmux attach -t session_name
```

- 强制接管会话：
```bash
tmux attach -d -t session_name
```

- 列出 session + 每个 pane 当前命令：
```bash
tmux list-panes -a -F "#{session_name}: #{pane_current_command}"
```

---

## 5️⃣ tmux 的优势

- **SSH 断线不影响任务**
- **可分屏同时监控训练/GPU/日志**
- **多会话管理方便**
- **训练日志可实时查看**

---

✅ **最小必学快捷键**：

- Ctrl+B D → detach
- Ctrl+B C → 新窗口
- Ctrl+B N / P → 切窗口
- Ctrl+B % → 垂直分屏
- Ctrl+B " → 水平分屏
- Ctrl+B 方向键 → 切分屏
- Ctrl+B Z → 放大 pane
- Ctrl+B  \[  →  查看历史输出

`tmux kill-session -t mysession`
    