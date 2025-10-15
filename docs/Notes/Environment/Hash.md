# Git 提交哈希完全指南

## 第一部分：基础概念

### 什么是提交哈希？

提交哈希是 Git 为每个提交自动生成的**唯一标识符**，使用 SHA-1 加密算法生成。

**特点：**
- 由 40 个十六进制字符组成（0-9, a-f）
- 每个提交都有唯一的哈希值
- 即使内容完全相同，提交时间或作者不同，哈希也会不同
- Git 内部通过哈希来追踪和管理所有提交

**示例：**
```
完整哈希：b42cbe5f3eb1d5158747283b26f92f8f22a28bb5
缩写哈希：b42cbe5
```

### 为什么需要提交哈希？

1. **唯一标识** - 在众多提交中精确定位某个提交
2. **操作指引** - 告诉 Git 你想对哪个提交进行操作
3. **版本管理** - 快速回溯到任意历史版本
4. **分支管理** - 从指定位置创建新分支

---

## 第二部分：获取提交哈希的详细方法

### 方法 1：`git log --oneline`（最推荐 - 简洁清晰）

**命令：**
```bash
git log --oneline
```

**输出示例：**
```
6c93c29 (HEAD -> e01) Example 01
8c95a41 (main, e02) Example 02
b42cbe5 BasicDemo
fcc1de0 initialization
```

**如何读取：**
```
6c93c29          ← 这是缩写哈希（7位）
(HEAD -> e01)    ← 当前位置和分支
Example 01       ← 提交信息
```

**优点：**
- 显示最简洁
- 一行一个提交
- 容易快速找到目标提交
- 包含分支信息

**获取哈希方式：** 直接复制左边的 7 位哈希

**常用变体：**
```bash
# 显示最近 5 个提交
git log --oneline -5

# 显示所有分支的提交
git log --oneline --all

# 显示特定文件的提交历史
git log --oneline -- <文件名>

# 显示指定作者的提交
git log --oneline --author="Yingguang"
```

---

### 方法 2：`git log --graph --all`（显示分支结构）

**命令：**
```bash
git log --graph --all
```

**输出示例：**
```
* commit 8c95a41fbc9cf02f4464efaaa3a497775f0e45a7 (main, e02)
| Author: John Developer <john.dev@example.com>
| Date:   Wed Oct 15 15:12:46 2025 +0100
| 
|     Example 02
|   
| * commit 6c93c298d56f28af4ae752ff4ff53b8a0f9d0f94 (HEAD -> e01)
|/  Author: John Developer <john.dev@example.com>
|   Date:   Wed Oct 15 13:39:41 2025 +0100
|   
|       Example 01
| 
* commit b42cbe5f3eb1d5158747283b26f92f8f22a28bb5
```

**如何读取：**
```
commit b42cbe5f3eb1d5158747283b26f92f8f22a28bb5
       ↑ 这是完整哈希（40位）
```

**优点：**
- 显示完整哈希（40 位）
- 显示分支结构和分叉
- 包含作者、日期等详细信息
- 适合理解提交历史关系

**获取哈希方式：** 复制 `commit` 后的完整哈希或前 7 位

**常用变体：**
```bash
# 显示最近 10 个提交的图形
git log --graph --all -10

# 简化显示，只显示哈希和信息
git log --graph --all --oneline

# 显示所有分支
git log --graph --decorate --all
```

---

### 方法 3：`git reflog`（查看操作历史）

**命令：**
```bash
git reflog
```

**输出示例：**
```
b42cbe5 (HEAD -> e01) HEAD@{0}: reset: moving to HEAD~1
6c93c29 HEAD@{1}: checkout: moving from main to e01
8c95a41 (main, e02) HEAD@{2}: commit: Example 02
b42cbe5 (HEAD -> e01) HEAD@{3}: reset: moving to HEAD~1
6c93c29 HEAD@{4}: commit: Example 01
b42cbe5 (HEAD -> e01) HEAD@{5}: commit: BasicDemo
fcc1de0 HEAD@{6}: commit (initial): initialization
```

**如何读取：**
```
b42cbe5          ← 哈希
HEAD@{0}         ← 引用位置（0 是最近的操作）
reset: moving    ← 执行的操作类型
```

**优点：**
- 显示所有 HEAD 的移动历史
- 包括 reset、checkout、commit 等所有操作
- 可以找回"丢失"的提交
- 每条记录都有时间顺序

**获取哈希方式：** 复制左边的哈希，或使用 `HEAD@{n}` 引用

**常用场景：**
```bash
# 恢复到 5 个操作之前的状态
git reset --hard HEAD@{5}

# 找回被 reset 的提交
git reflog
# 找到想要的哈希后
git checkout <哈希>
```

---

### 方法 4：`git log` 完整版（所有详细信息）

**命令：**
```bash
git log
```

**输出示例：**
```
commit 8c95a41fbc9cf02f4464efaaa3a497775f0e45a7
Author: YINGGUNG XING <yingguang.xing@student.griffith.ie>
Date:   Wed Oct 15 15:12:46 2025 +0100

    Example 02
```

**优点：**
- 显示完整的提交信息
- 包含完整哈希、作者、日期、完整提交信息

**缺点：**
- 显示冗长
- 一次只显示一个提交

**常用变体：**
```bash
# 显示最近 5 个提交
git log -5

# 显示指定提交的详细信息
git log -1 <哈希>

# 显示提交的详细变化
git log -p

# 显示统计信息
git log --stat
```

---

### 方法 5：`git show` 查看单个提交

**命令：**
```bash
git show <哈希>
```

**示例：**
```bash
git show b42cbe5
```

**输出：** 显示该提交的完整信息和代码变化

**优点：**
- 清楚看到提交做了什么改变
- 显示diff信息
- 可以用来验证是否是想要的提交

---

## 第三部分：哈希的三种表示方式

### 表示方式 1：完整哈希（40 位）

**格式：**
```
b42cbe5f3eb1d5158747283b26f92f8f22a28bb5
```

**使用场景：**
- 需要绝对精确的时候
- 复制粘贴完整哈希时
- 在文档中记录时

**用法示例：**
```bash
git switch -c e03 b42cbe5f3eb1d5158747283b26f92f8f22a28bb5
git reset --hard b42cbe5f3eb1d5158747283b26f92f8f22a28bb5
```

---

### 表示方式 2：缩写哈希（7 位，通常）

**格式：**
```
b42cbe5
```

**说明：**
- Git 会自动截断到足以唯一识别的长度
- 通常 7 位就够了
- 可以写更少（如 5 位），只要不冲突

**使用场景：**
- 日常命令操作中（最常用）
- 快速输入
- Git 命令中默认显示的格式

**用法示例：**
```bash
git switch -c e03 b42cbe5        # 最常用
git switch -c e03 b42c           # 也可以
git show b42cbe5
git reset --hard b42cbe5
```

**如何确定最短有效长度：**
```bash
# Git 会自动用最短的唯一标识
git log --abbrev-commit --oneline  # 显示默认长度
git log --abbrev=5 --oneline       # 指定 5 位
```

---

### 表示方式 3：相对引用（相对位置）

**格式和含义：**
```bash
HEAD~1    # 当前 HEAD 的前 1 个提交
HEAD~2    # 当前 HEAD 的前 2 个提交
HEAD~3    # 当前 HEAD 的前 3 个提交
HEAD^     # 同 HEAD~1
HEAD^^    # 同 HEAD~2
```

**可视化例子：**
```
HEAD      ← 当前位置（Example 01）
  ↑
HEAD~1    ← 往前 1 个（BasicDemo）
  ↑
HEAD~2    ← 往前 2 个（initialization）
```

**使用场景：**
- 回退到最近的几个提交
- 操作时不需要记住哈希

**用法示例：**
```bash
# 回退 1 个提交
git reset --hard HEAD~1

# 从前 2 个提交创建分支
git switch -c bugfix HEAD~2

# 查看前 3 个提交
git log HEAD~3
```

**与分支结合：**
```bash
# 从 main 的前 1 个提交创建分支
git switch -c feature main~1

# 显示 e01 分支的前 2 个提交
git log e01~2
```

---

## 第四部分：提交哈希的实际应用场景

### 应用 1：从指定提交创建新分支

**场景：** 你想从 BasicDemo 分出一条新分支来开发新功能

**步骤：**
```bash
# 1. 查看哈希
git log --oneline
# 输出：b42cbe5 BasicDemo

# 2. 从 BasicDemo 创建新分支 e03
git switch -c e03 b42cbe5

# 3. 验证
git branch
# 输出：
#   e01
#   e02
#   main
# * e03  ← 你现在在这里
```

**完整代码变化：**
```
Before:
* 8c95a41 (main, e02) - Example 02
|
| * 6c93c29 (e01) - Example 01
|/
* b42cbe5 - BasicDemo

After:
* 8c95a41 (main, e02) - Example 02
|
| * 6c93c29 (e01) - Example 01
|/
|
| * HEAD -> e03（你现在在这里）
|/
* b42cbe5 - BasicDemo
```

---

### 应用 2：恢复已删除或已 reset 的提交

**场景：** 执行了 `git reset --hard HEAD~1` 后，想恢复被删除的提交

**步骤：**
```bash
# 1. 查看操作历史
git reflog
# 输出：
# b42cbe5 HEAD@{0}: reset: moving to HEAD~1
# 6c93c29 HEAD@{1}: checkout: moving from main to e01  ← 想要恢复到这里
# ...

# 2. 恢复到想要的状态
git reset --hard 6c93c29
# 或
git reset --hard HEAD@{1}

# 3. 验证
git log --oneline
# 6c93c29 (HEAD -> e01) Example 01  ← 恢复了！
# b42cbe5 BasicDemo
# ...
```

---

### 应用 3：检出到特定提交（分离头指针）

**场景：** 你想查看 BasicDemo 时的代码状态

**步骤：**
```bash
# 1. 检出到 BasicDemo 提交
git switch b42cbe5
# 或
git checkout b42cbe5

# 2. 查看当前状态
git status
# 输出：HEAD detached at b42cbe5

# 3. 查看代码
# 此时工作目录是 BasicDemo 时的样子

# 4. 如果想创建分支保存工作
git switch -c temp-branch

# 5. 返回主分支
git switch main
```

---

### 应用 4：比较两个提交之间的差异

**场景：** 想看 Example 01 和 BasicDemo 之间改了什么

**步骤：**
```bash
# 1. 查看哈希
git log --oneline
# 6c93c29 Example 01
# b42cbe5 BasicDemo

# 2. 比较两个提交
git diff b42cbe5 6c93c29

# 或显示统计信息
git diff --stat b42cbe5 6c93c29
```

---

### 应用 5：cherry-pick（挑选特定提交）

**场景：** 你只想把 e01 分支的 Example 01 提交复制到 main 分支，不要 BasicDemo

**步骤：**
```bash
# 1. 查看 e01 的提交哈希
git log e01 --oneline
# 6c93c29 Example 01

# 2. 切换到 main
git switch main

# 3. cherry-pick 那个提交
git cherry-pick 6c93c29

# 4. 验证
git log --oneline
# 会看到 Example 01 被复制到 main 上
```

---

### 应用 6：标记重要提交

**场景：** BasicDemo 是个里程碑，想给它取个好记的名字

**步骤：**
```bash
# 1. 为提交创建标签
git tag v1.0 b42cbe5

# 2. 以后可以用标签代替哈希
git switch -c release v1.0

# 3. 查看所有标签
git tag
```

---

## 第五部分：实用技巧和最佳实践

### 技巧 1：快速复制哈希

**方法：**
```bash
# 使用 git log 并管道输出
git log --oneline | head -1 | awk '{print $1}'

# 或在命令行上下文中
git log -1 --pretty=format:"%H"  # 完整哈希
git log -1 --pretty=format:"%h"  # 缩写哈希
```

---

### 技巧 2：创建便于记忆的别名

**配置：**
```bash
# 创建简短的别名
git config --global alias.lga 'log --graph --all --oneline'

# 然后可以用
git lga
# 代替 git log --graph --all --oneline
```

---

### 技巧 3：快速对比提交

**常用命令：**
```bash
# 查看从 A 到 B 改了什么
git diff <哈希A> <哈希B>

# 查看某个提交改了什么
git show <哈希>

# 查看两个分支的差异
git diff main e01
```

---

### 技巧 4：安全的 reset 操作

**最佳实践：**
```bash
# 1. 先看看要改什么
git log --oneline -5

# 2. 用 soft reset 先看看结果（不删除工作目录）
git reset --soft <哈希>

# 3. 检查没问题再用 hard
git reset --hard <哈希>

# 4. 如果后悔了
git reflog
git reset --hard <之前的哈希>
```

---

### 技巧 5：记住重要的相对位置

**常用场景：**
```bash
# 回退到前一个提交
git reset --hard HEAD~1

# 从前两个提交创建分支
git branch feature HEAD~2

# 比较最后两个提交的差异
git diff HEAD~1 HEAD
```

---

## 第六部分：常见问题和解决方案

### 问题 1：哈希太长，容易打错

**解决方案：**
- 使用 7 位缩写哈希（足够唯一）
- 或使用相对引用 `HEAD~n`
- 或复制粘贴

---

### 问题 2：找不到需要的提交哈希

**解决方案：**
```bash
# 查看所有历史
git reflog

# 搜索特定提交信息
git log --oneline --grep="关键词"

# 搜索特定作者的提交
git log --oneline --author="名字"
```

---

### 问题 3：哈希冲突（两个不同提交有相似哈希）

**说明：** 几乎不会发生，但如果发生：
```bash
# 使用更长的哈希
git switch -c branch 8c95a41f  # 使用 8 位或更多

# 或使用完整哈希
git switch -c branch 8c95a41fbc9cf02f4464efaaa3a497775f0e45a7
```

---

### 问题 4：想看某个哈希对应的分支

**解决方案：**
```bash
# 查看哪些分支指向这个提交
git branch -a --contains <哈希>

# 或用 log
git log --oneline --decorate
```

---

## 总结速记表

| 需求 | 命令 | 说明 |
|------|------|------|
| **查看所有提交** | `git log --oneline` | 最常用，简洁清晰 |
| **看分支结构** | `git log --graph --all` | 显示分支分叉关系 |
| **查看操作历史** | `git reflog` | 可以找回删除的提交 |
| **创建新分支** | `git switch -c <名> <哈希>` | 从指定提交创建 |
| **切换到提交** | `git switch <哈希>` | 分离头指针状态 |
| **恢复提交** | `git reset --hard <哈希>` | 回退到指定提交 |
| **比较提交** | `git diff <哈希1> <哈希2>` | 查看差异 |
| **查看提交详情** | `git show <哈希>` | 显示提交的所有信息 |
| **复制提交** | `git cherry-pick <哈希>` | 把提交复制到当前分支 |
| **相对引用** | `HEAD~1`、`HEAD~2` | 往前数几个提交 |

---

## 实战工作流总结

```bash
# ❶ 查看提交历史
git log --oneline

# ❷ 找到目标提交哈希
# 例如：b42cbe5 BasicDemo

# ❸ 执行操作
git switch -c e03 b42cbe5      # 从这里创建新分支
# 或
git reset --hard b42cbe5        # 回退到这个提交
# 或
git cherry-pick b42cbe5         # 复制这个提交

# ❹ 验证结果
git log --oneline
git status

# ❺ 如果出错了，查看历史
git reflog

# ❻ 恢复
git reset --hard <之前的哈希或HEAD@{n}>
```