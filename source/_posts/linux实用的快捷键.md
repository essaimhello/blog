---
title: linux实用的快捷键
tags: []
categories: Linux运维
author: Judy
cover: /images/default-cover.jpg
published: true
date: 2026-01-23 17:53:05
---

# {{ title }}
> 本文首发于我的个人博客，转载请注明出处。

### 一、4 个核心快捷键（终端文本编辑 / 控制）
| 快捷键 | 功能说明（Bash/Zsh 通用） |
| --- | --- |
| `Ctrl + A` | **光标跳转到行首**（A = Beginning）<br>例：输入 `echo "hello"` 后按 `Ctrl+A`，光标会到 `e` 前面 |
| `Ctrl + E` | **光标跳转到行尾**（E = End）<br>例：输入 `echo "hello"` 后按 `Ctrl+E`，光标会到 `"` 后面 |
| `Ctrl + D` | 1. **删除光标所在位置的字符**（类似 `Delete` 键）；<br>2. 若光标在空行，按则 **退出当前终端 / Session**（等价于 `exit`） |
| `Ctrl + W` | **删除光标左侧到最近空格的单词**（W = Word）<br>例：输入 `git push origin main` ，光标在 `main` 后，按 `Ctrl+W` 会删除 `main`；再按会删除 `origin` |

### 二、高频补充快捷键（按场景分类）
#### 1. 文本编辑类（修改命令行输入）
| 快捷键 | 功能说明 |
| --- | --- |
| `Ctrl + K` | 删除光标到行尾的所有字符（K = Kill） |
| `Ctrl + U` | 删除光标到行首的所有字符（U = Undo Line） |
| `Ctrl + Y` | 粘贴之前用 `Ctrl+K/Ctrl+U/Ctrl+W` 删除的内容（Y = Yank） |
| `Ctrl + H` | 删除光标左侧的字符（类似 `Backspace` 键） |
| `Ctrl + T` | 交换光标所在字符和前一个字符（T = Swap）<br>例：输入 `teh` 后按 `Ctrl+T` 变成 `the` |
| `Alt + D` | 删除光标右侧到最近空格的单词（Alt+D = Delete Word Right） |
| `Alt + B` | 光标向左跳一个单词（B = Back Word） |
| `Alt + F` | 光标向右跳一个单词（F = Forward Word） |

#### 2. 终端控制类（会话 / 进程管理）
| 快捷键 | 功能说明 |
| --- | --- |
| `Ctrl + C` | 强制终止当前运行的进程（最常用！）<br>例：`ping baidu.com` 时按 `Ctrl+C` 停止 |
| `Ctrl + Z` | 暂停当前进程，将其放入后台（Z = Suspend）<br>后续可通过 `fg` 恢复到前台、`bg` 让其在后台运行 |
| `Ctrl + L` | 清屏（等价于 `clear` 命令，L = Clear） |
| `Ctrl + S` | 暂停终端输出（防止日志刷屏，S = Stop） |
| `Ctrl + Q` | 恢复终端输出（解除 `Ctrl+S` 的暂停，Q = Quit Stop） |
| `Ctrl + R` | 搜索历史命令（R = Reverse Search）<br>按后输入关键词，会匹配之前执行过的命令，按 `Ctrl+R` 切换下一个匹配项 |

#### 3. 历史命令类（快速复用之前的命令）
| 快捷键 | 功能说明 |
| --- | --- |
| `↑` / `↓` | 上下切换历史命令 |
| `Ctrl + P` | 上一条历史命令（等价于 `↑`，P = Previous） |
| `Ctrl + N` | 下一条历史命令（等价于 `↓`，N = Next） |
| `!!` | 执行上一条命令（终端输入 `!!` 回车即可） |
| `!n` | 执行第 n 条历史命令（`history` 命令可查看序号） |
| `!keyword` | 执行最近一条以 `keyword` 开头的命令<br>例：`!git` 会执行上一条 `git` 开头的命令 | 