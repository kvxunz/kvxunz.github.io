# Git 核心原理与实战笔记

> 简洁、清晰、系统的 Git 学习指南
>
> 更新日期：2025-10-13

---

## 目录

[TOC]

---

## 第一部分：核心原理

### 1. Git 是什么？

Git 是一个**分布式版本控制系统**，核心思想：

```
游戏存档系统 = Git 版本控制
每次 commit = 一个完整的游戏存档
可以随时读档 = 回到任何历史版本
```

**三大特点：**
1. **快照存储**：每次提交保存完整快照，不是差异
2. **内容寻址**：通过 SHA-1 哈希值识别内容，相同内容只存一份
3. **轻量分支**：分支只是指向提交的指针，创建和切换都很快

---

### 2. Git 的数据存储原理

#### 2.1 三种核心对象

```
Blob (文件内容)
  ↓
Tree (目录结构)
  ↓
Commit (提交记录)
```

| 对象 | 作用 | 内容 |
|------|------|------|
| **Blob** | 存储文件内容 | 文件的二进制数据 |
| **Tree** | 存储目录结构 | 文件名 + Blob 引用 |
| **Commit** | 存储提交信息 | 作者、时间、Tree 引用 |

#### 2.2 哈希值（SHA-1）

```bash
# Git 对文件内容计算哈希值
文件内容 "Hello" → SHA-1 → abc123def456...

# 相同内容 = 相同哈希值 = 只存储一份
文件 A: "Hello" → abc123
文件 B: "Hello" → abc123（复用，不重复存储）
```

---

### 3. Git 的四个工作区

这是 Git 最核心的概念，理解了它就理解了 Git。

```
┌─────────────┐  git add   ┌─────────────┐  git commit  ┌─────────────┐  git push  ┌─────────────┐
│   工作区     │ ────────→  │   暂存区     │  ─────────→  │  本地仓库    │ ─────────→ │  远程仓库    │
│  Working    │            │  Staging    │             │   Local     │            │   Remote    │
│  Directory  │ ←────────  │    Index    │  ←────────  │  Repository │ ←──────── │  Repository │
└─────────────┘ git restore└─────────────┘ git reset   └─────────────┘  git pull  └─────────────┘
   你正在编辑              准备提交的内容            本地历史记录              服务器备份
```

#### 3.1 工作区（Working Directory）

**定义：** 项目文件夹中你能看到和编辑的文件（除了 `.git/`）

**特点：**
- 直接可见可编辑
- 修改容易丢失
- 还没有被 Git "正式记录"

**文件状态：**
- `Untracked`：新文件，Git 不认识
- `Modified`：已修改，但未添加到暂存区
- `Deleted`：已删除

**操作原理：**
```bash
# 编辑文件
vim index.html

# Git 通过对比文件的哈希值，发现文件被修改
git status  # Git 提示: modified: index.html
```

---

#### 3.2 暂存区（Staging Area / Index）

**定义：** `.git/index` 文件，临时存放准备提交的内容

**为什么需要暂存区？精确控制每次提交的内容！**

**场景示例：**
```bash
# 你同时修改了 5 个文件
修改：login.js, auth.js, user.js, style.css, README.md

# 但这是两个不同的功能
功能 A: login.js, auth.js, user.js  → 登录功能
功能 B: style.css, README.md       → 样式优化

# 分两次提交
git add login.js auth.js user.js
git commit -m "新增登录功能"

git add style.css README.md
git commit -m "优化页面样式"
```

**操作原理：**
```bash
# git add 的工作原理
1. 计算文件的 SHA-1 哈希值
2. 将文件内容压缩存储到 .git/objects/
3. 在 .git/index 中记录文件名和哈希值的映射

# 举例
git add index.html
→ 计算 SHA-1: a1b2c3d4...
→ 存储到: .git/objects/a1/b2c3d4...
→ 更新索引: index.html → a1b2c3d4
```

---

#### 3.3 本地仓库（Local Repository）

**定义：** `.git/` 目录，存储所有历史版本

**结构：**
```
.git/
├── objects/        ← 所有版本的文件内容（压缩存储）
├── refs/           ← 分支和标签的指针
│   ├── heads/      ← 本地分支（main, dev, etc.）
│   └── remotes/    ← 远程分支的本地副本
├── HEAD            ← 指向当前所在的分支
├── index           ← 暂存区
└── config          ← 仓库配置
```

**操作原理：**
```bash
# git commit 的工作原理
1. 基于暂存区的内容创建 Tree 对象
2. 创建 Commit 对象（包含作者、时间、父提交等）
3. 将当前分支指针指向新的 Commit

# 举例
git commit -m "Add login"
→ 创建 Tree 对象: t7h8i9j0
→ 创建 Commit 对象: c4d5e6f7
→ 更新 main 分支指针: main → c4d5e6f7
```

---

#### 3.4 远程仓库（Remote Repository）

**定义：** 服务器上的 Git 仓库（GitHub/GitLab/Gitee）

**作用：**
- 团队协作：多人共享代码
- 备份：防止本地代码丢失
- 发布：让其他人使用你的代码

**操作原理：**
```bash
# git push 的工作原理
1. 比对本地和远程的提交历史
2. 如果本地有新提交，打包发送到服务器
3. 更新远程分支指针

# git pull 的工作原理
git pull = git fetch + git merge
1. fetch: 下载远程的提交到本地
2. merge: 合并到当前分支
```

---

### 4. 工作区关系图

#### 4.1 数据流转图

```
         修改文件                添加到暂存区              提交到仓库               推送到远程
┌──────┐        ┌──────┐        ┌──────┐        ┌──────┐        ┌──────┐
│ 未修改│ ────→ │ 已修改│ ────→ │ 已暂存│ ────→ │ 已提交│ ────→ │ 已推送│
└──────┘  edit  └──────┘ git add└──────┘git commit└──────┘git push└──────┘
                     │               │               │
                     │ git restore   │ git restore   │ git reset
                     │  (撤销)       │  --staged     │  (回退)
                     ↓               ↓               ↓
                 恢复原样        移出暂存区      撤销提交
```

#### 4.2 命令对应关系

| 方向 | 命令 | 作用范围 | 原理 |
|------|------|---------|------|
| **→** | `git add` | 工作区 → 暂存区 | 计算哈希，存储对象，更新索引 |
| **→** | `git commit` | 暂存区 → 本地仓库 | 创建 Commit 对象，移动分支指针 |
| **→** | `git push` | 本地仓库 → 远程仓库 | 发送新提交，更新远程分支 |
| **←** | `git restore <file>` | 暂存区 → 工作区 | 用暂存区内容覆盖工作区 |
| **←** | `git restore --staged` | 本地仓库 → 暂存区 | 更新索引，指向 HEAD 的版本 |
| **←** | `git reset` | 移动 HEAD 指针 | 改变分支指向的提交 |
| **←** | `git pull` | 远程仓库 → 本地仓库 | fetch + merge |

---

## 第二部分：基础操作

### 1. 初始化和配置

```bash
# 初始化仓库
git init                              # 在当前目录创建 .git/
git clone <url>                       # 克隆远程仓库

# 配置用户信息（必须）
git config --global user.name "Name"
git config --global user.email "email@example.com"

# 配置默认分支
git config --global init.defaultBranch main

# 查看配置
git config --list
```

**原理：**
- `--global`：配置保存在 `~/.gitconfig`（全局配置）
- 不加 `--global`：配置保存在 `.git/config`（仓库配置）

---

### 2. 查看状态和差异

```bash
# 查看工作区状态
git status                    # 查看哪些文件被修改、添加、删除

# 查看差异
git diff                      # 工作区 vs 暂存区
git diff --staged             # 暂存区 vs 本地仓库（HEAD）
git diff HEAD                 # 工作区 vs 本地仓库
git diff <commit1> <commit2>  # 两个提交之间的差异
```

**原理：**
```bash
git diff 的工作流程：
1. 读取两个版本的文件内容
2. 对比文件的每一行
3. 显示差异（+ 表示新增，- 表示删除）
```

---

### 3. 添加和提交

```bash
# 添加到暂存区
git add <file>                # 添加指定文件
git add .                     # 添加所有修改
git add *.js                  # 添加所有 .js 文件

# 部分添加（交互式）
git add -p <file>             # 逐个确认每个修改块

# 提交到本地仓库
git commit -m "message"       # 提交并附加信息
git commit                    # 打开编辑器写详细信息
git commit -am "message"      # add + commit（仅对已跟踪文件）

# 修改最后一次提交
git commit --amend            # 修改提交信息或添加文件
```

**git commit 的完整流程：**
```bash
1. 读取暂存区的内容
2. 创建 Tree 对象（记录目录结构）
3. 创建 Commit 对象：
   - tree: 指向 Tree 对象
   - parent: 指向父提交
   - author: 作者信息
   - committer: 提交者信息
   - message: 提交信息
4. 移动当前分支指针到新的 Commit
5. 清空暂存区
```

---

### 4. 查看历史

```bash
# 查看提交历史
git log                           # 详细历史
git log --oneline                 # 简洁模式（一行一个提交）
git log --graph --all             # 图形化显示所有分支
git log -p                        # 显示每次提交的详细差异
git log --stat                    # 显示统计信息

# 查看指定文件的历史
git log <file>                    # 文件的提交历史
git log -p <file>                 # 文件的详细修改历史

# 查看某次提交的详情
git show <commit>                 # 显示提交的详细信息

# 查看操作历史（包括被删除的提交）
git reflog                        # 记录所有 HEAD 的移动
```

**git log 的原理：**
```bash
从当前 HEAD 开始，沿着 parent 指针往回遍历：
HEAD → Commit C → Commit B → Commit A
```

---

## 第三部分：分支管理

### 1. 分支的本质

**分支是什么？** 一个指向某个 Commit 的指针（只有 41 字节！）

```bash
# 分支的存储位置
.git/refs/heads/main     # 内容：指向某个 commit 的 SHA-1
.git/refs/heads/dev      # 内容：指向某个 commit 的 SHA-1

# HEAD 指针
.git/HEAD                # 内容：ref: refs/heads/main
```

**分支图示：**
```
         ┌─ dev (指针)
         ↓
A ← B ← C ← D
         ↑
         └─ main (指针)
         
HEAD → main (当前在 main 分支)
```

---

### 2. 分支基本操作

```bash
# 查看分支
git branch                # 查看本地分支
git branch -r             # 查看远程分支
git branch -a             # 查看所有分支

# 创建分支
git branch <name>         # 创建分支（不切换）
git checkout -b <name>    # 创建并切换（旧命令）
git switch -c <name>      # 创建并切换（新命令，推荐）

# 切换分支
git checkout <name>       # 切换分支（旧命令）
git switch <name>         # 切换分支（新命令）

# 删除分支
git branch -d <name>      # 删除已合并的分支
git branch -D <name>      # 强制删除分支

# 重命名分支
git branch -m <old> <new> # 重命名分支
```

**切换分支的原理：**
```bash
git switch dev 的流程：
1. 更新 HEAD 指向 dev 分支
2. 更新暂存区为 dev 分支的快照
3. 更新工作区文件为 dev 分支的状态
```

---

### 3. 分支合并

#### 3.1 Fast-Forward（快进合并）

**场景：** 当前分支没有新提交，直接移动指针

```bash
# 操作
git checkout main
git merge dev

# 图示
合并前：
main → A ← B ← C
              ↓
             dev → D ← E

合并后：
main, dev → A ← B ← C ← D ← E
```

**原理：** 直接移动 main 指针到 dev 所指向的提交

---

#### 3.2 Three-Way Merge（三方合并）

**场景：** 当前分支有新提交，创建合并提交

```bash
# 操作
git checkout main
git merge dev

# 图示
合并前：
        C (main)          E (dev)
        ↓                 ↓
A ← B ← C ← F             D ← E
        ↓                 ↓
      (共同祖先 B)

合并后：
        C ← F ─────┐
        ↓           ↓
A ← B ← C ← F ← M (合并提交)
        ↓       ↑
        D ← E ──┘
```

**原理：**
```bash
1. 找到共同祖先 B
2. 对比三个版本：共同祖先 B、main 的 F、dev 的 E
3. 自动合并没有冲突的部分
4. 有冲突的部分标记出来，等待手动解决
5. 创建新的合并提交 M（有两个父提交）
```

---

### 4. Rebase（变基）

**作用：** 让提交历史更线性

```bash
# 操作
git checkout dev
git rebase main

# 对比 Merge vs Rebase
Merge:
A ← B ← C ← F ─────┐
        ↓           ├→ M
        D ← E ──────┘

Rebase:
A ← B ← C ← F ← D' ← E'
（D 和 E 被"移动"到 F 之后）
```

**原理：**
```bash
git rebase main 的流程：
1. 找到当前分支和 main 的共同祖先
2. 提取当前分支的所有提交（D、E）
3. 将当前分支指向 main
4. 依次应用之前提取的提交（创建新的 D'、E'）
5. 原来的 D、E 变成"悬空"提交（可以用 reflog 找回）
```

**⚠️ 黄金法则：永远不要 rebase 已推送的提交！**

---

## 第四部分：撤销操作

这是 Git 中最容易混淆的部分，核心是理解 `reset` 的三种模式。

### 1. 撤销工作区修改

**场景：** 修改了文件，但还没 `git add`

```bash
# 撤销单个文件
git restore <file>            # 新命令（推荐）
git checkout -- <file>        # 旧命令

# 撤销所有修改
git restore .
```

**原理：**
```bash
用暂存区（或 HEAD）的内容覆盖工作区文件
工作区的修改被永久丢弃（无法找回！）
```

---

### 2. 撤销暂存区

**场景：** 已经 `git add`，但还没 `git commit`

```bash
# 从暂存区移除（但保留工作区修改）
git restore --staged <file>   # 新命令
git reset HEAD <file>          # 旧命令
```

**原理：**
```bash
将暂存区的文件指针重置为 HEAD 版本
工作区的修改保留
```

---

### 3. 撤销提交（核心）

**场景：** 已经 `git commit`，想撤销

#### 3.1 git reset 三种模式

```bash
git reset --soft HEAD~1       # 模式 1：温柔回退
git reset --mixed HEAD~1      # 模式 2：默认模式
git reset --hard HEAD~1       # 模式 3：暴力回退
```

**三种模式对比表：**

| 模式 | 本地仓库<br>(HEAD) | 暂存区<br>(Index) | 工作区<br>(Working) | 适用场景 |
|------|:---:|:---:|:---:|----------|
| `--soft` | ✅ 回退 | ❌ 保留 | ❌ 保留 | 修改提交信息<br>拆分提交 |
| `--mixed`<br>(默认) | ✅ 回退 | ✅ 清空 | ❌ 保留 | 重新组织暂存区<br>修改后再提交 |
| `--hard` | ✅ 回退 | ✅ 清空 | ✅ 清空 | 完全放弃修改<br>⚠️ 危险！ |

---

#### 3.2 详细原理

**场景假设：**
```bash
初始状态：
A ← B ← C ← D (HEAD → main)
        
工作区：   index.html (修改了)
暂存区：   index.html (已 add)
本地仓库： D 提交（包含 index.html）
```

**执行 `git reset --soft HEAD~1` 后：**
```bash
A ← B ← C (HEAD → main)  # HEAD 回退到 C
        D (悬空提交，可以用 reflog 找回)

工作区：   index.html (修改了) ✅ 保留
暂存区：   index.html (已 add) ✅ 保留
本地仓库： C 提交

可以做什么？
→ git commit -m "新的提交信息"（修改提交信息）
→ 继续修改后再提交
```

**执行 `git reset --mixed HEAD~1` 后：**
```bash
A ← B ← C (HEAD → main)

工作区：   index.html (修改了) ✅ 保留
暂存区：   (空) ❌ 清空
本地仓库： C 提交

可以做什么？
→ git add index.html（重新添加）
→ git commit（重新提交）
```

**执行 `git reset --hard HEAD~1` 后：**
```bash
A ← B ← C (HEAD → main)

工作区：   index.html (恢复成 C 的状态) ❌ 清空
暂存区：   (空) ❌ 清空
本地仓库： C 提交

⚠️ 警告：所有未提交的修改永久丢失！
```

---

#### 3.3 reset 的指针操作

```bash
# HEAD 的多种指向方式
HEAD       # 当前提交
HEAD~1     # 上一个提交
HEAD~2     # 上上个提交
HEAD^      # 上一个提交（等同于 HEAD~1）
HEAD^^     # 上上个提交

# 用 commit hash
git reset --soft abc1234

# 回退 3 个提交
git reset --soft HEAD~3
```

**原理：**
```bash
git reset 的核心操作：
1. 移动 HEAD 指针（以及当前分支指针）
2. 根据模式，决定是否更新暂存区和工作区

示例：
初始：HEAD → main → D
执行 git reset --soft C
结果：HEAD → main → C（D 变成悬空提交）
```

---

### 4. revert（反向提交）

**场景：** 已经 `git push`，想撤销某个提交

```bash
# 创建一个新提交来撤销指定提交
git revert <commit>

# 撤销最后一次提交
git revert HEAD
```

**原理：**
```bash
假设历史：A ← B ← C ← D

git revert C 的流程：
1. 计算 C 的反向修改（C 改了什么，就反着改回去）
2. 创建新的提交 C'，应用反向修改
3. 结果：A ← B ← C ← D ← C'

注意：
- C 提交仍然在历史中
- 历史记录没有被改写
- 可以安全地推送到远程
```

**reset vs revert：**
```bash
reset（重写历史）：
A ← B ← C ← D
执行 git reset --hard B
A ← B（C 和 D 消失，但可以用 reflog 找回）

revert（新建提交）：
A ← B ← C ← D
执行 git revert C
A ← B ← C ← D ← C'（C' 撤销 C 的修改）
```

---

## 第五部分：远程协作

### 1. 远程仓库基础

```bash
# 查看远程仓库
git remote -v                         # 查看远程仓库 URL
git remote show origin                # 查看详细信息

# 添加远程仓库
git remote add origin <url>           # origin 是默认名称

# 修改远程仓库
git remote set-url origin <new-url>   # 修改 URL
git remote rename origin upstream     # 重命名
git remote remove origin              # 删除
```

---

### 2. 推送和拉取

#### 2.1 git push（推送）

```bash
# 推送到远程
git push origin main                  # 推送 main 分支
git push -u origin main               # 首次推送，建立追踪关系
git push                              # 推送当前分支（需要先建立追踪）

# 推送所有分支
git push --all

# 强制推送（⚠️ 危险）
git push -f origin main
git push --force-with-lease origin main  # 更安全的强制推送
```

**git push 的原理：**
```bash
1. 比对本地和远程的提交历史
2. 如果远程有本地没有的提交 → 拒绝推送（non-fast-forward）
3. 如果本地有新提交 → 打包发送到远程服务器
4. 更新远程分支指针

示例：
本地：  A ← B ← C ← D (main)
远程：  A ← B ← C (origin/main)
推送后：A ← B ← C ← D (origin/main)
```

---

#### 2.2 git pull（拉取）

```bash
# 拉取并合并
git pull origin main                  # 拉取 main 分支

# 等价于
git fetch origin
git merge origin/main

# 拉取并 rebase
git pull --rebase origin main
```

**git pull 的原理：**
```bash
git pull = git fetch + git merge

步骤：
1. fetch: 下载远程的新提交
2. merge: 合并到当前分支

示例：
本地：  A ← B ← C ← E (main)
远程：  A ← B ← C ← D (origin/main)

执行 git pull：
1. fetch 下载 D
2. merge 创建合并提交 M
结果：  A ← B ← C ← E ─┐
                  ↓     ├→ M (main)
                  D ────┘
```

---

#### 2.3 git fetch（获取）

```bash
# 下载远程更新（不合并）
git fetch origin                      # 获取所有分支
git fetch origin main                 # 只获取 main 分支

# 查看远程分支
git branch -r                         # 远程分支列表
git log origin/main                   # 查看远程分支的历史
```

**git fetch 的原理：**
```bash
只下载，不合并：
1. 从远程服务器下载新的提交
2. 更新 .git/refs/remotes/origin/* 指针
3. 不修改本地分支和工作区

本地：  A ← B ← C (main)
远程：  A ← B ← C ← D (origin/main)

执行 git fetch：
本地：  A ← B ← C (main)
        A ← B ← C ← D (origin/main，本地副本)

可以先查看，再决定是否合并：
git log origin/main
git merge origin/main
```

---

### 3. 冲突解决

#### 3.1 冲突产生的原因

```bash
场景：你和同事都修改了同一个文件的同一位置

你的修改：    同事的修改：
A ← B ← C ← E   A ← B ← C ← D
        ↑
    共同祖先 C

当你 git pull 时，Git 尝试合并 D 和 E，发现冲突
```

---

#### 3.2 冲突标记

```html
<!DOCTYPE html>
<html>
<head>
<<<<<<< HEAD
    <title>我的标题</title>    ← 你的版本
=======
    <title>同事的标题</title>  ← 远程的版本
>>>>>>> origin/main
</head>
</html>
```

**标记说明：**
- `<<<<<<< HEAD`：当前分支的修改开始
- `=======`：分隔线
- `>>>>>>> origin/main`：远程分支的修改结束

---

#### 3.3 解决冲突的完整流程

```bash
# 1. 拉取远程代码（发现冲突）
git pull origin main
# 输出：CONFLICT (content): Merge conflict in index.html

# 2. 查看冲突文件
git status
# 输出：
# Unmerged paths:
#   both modified:   index.html

# 3. 打开文件，手动解决冲突
vim index.html
# 删除冲突标记，保留想要的内容

# 4. 标记冲突已解决
git add index.html

# 5. 完成合并
git commit
# 或
git commit -m "解决冲突：保留新标题"

# 6. 推送
git push origin main
```

**快速选择某一方的版本：**
```bash
# 保留本地版本（我的版本）
git checkout --ours index.html
git add index.html

# 保留远程版本（他们的版本）
git checkout --theirs index.html
git add index.html
```

**冲突解决的原理：**
```bash
Git 的三方合并：
1. 找到共同祖先（base）
2. 对比你的版本（ours）
3. 对比远程版本（theirs）
4. 自动合并不冲突的部分
5. 冲突的部分标记出来，等待手动解决
6. 解决后创建合并提交
```

---

### 4. 协作最佳实践

**完整的团队协作流程：**

```bash
# 第 1 步：开始工作前，拉取最新代码
git checkout main
git pull origin main

# 第 2 步：创建功能分支
git checkout -b feature/user-login

# 第 3 步：开发功能
# ... 编写代码 ...
git add .
git commit -m "feat: 添加用户登录功能"

# 第 4 步：合并前再次同步主分支
git checkout main
git pull origin main

# 第 5 步：合并功能分支
git merge feature/user-login

# 第 6 步：处理冲突（如果有）
# ... 解决冲突 ...

# 第 7 步：推送
git push origin main

# 第 8 步：删除功能分支
git branch -d feature/user-login
```

---

## 第六部分：高级操作

### 1. Git Stash（暂存工作）

**场景：** 正在开发功能，突然要修复紧急 bug

```bash
# 保存当前工作
git stash                             # 保存工作区和暂存区
git stash save "正在开发登录功能"     # 带描述的保存

# 查看所有 stash
git stash list
# 输出：
# stash@{0}: On main: 正在开发登录功能
# stash@{1}: WIP on dev: abc1234 修复 bug

# 恢复 stash
git stash pop                         # 恢复最近的 stash 并删除
git stash apply                       # 恢复但不删除
git stash apply stash@{1}             # 恢复指定的 stash

# 删除 stash
git stash drop stash@{0}              # 删除指定 stash
git stash clear                       # 清空所有 stash
```

**git stash 的原理：**
```bash
1. 将工作区和暂存区的修改保存到一个特殊的提交
2. 恢复工作区到干净状态（相当于 git reset --hard）
3. stash 存储在 .git/refs/stash 中

内部结构：
stash@{0} → Commit (包含工作区和暂存区的快照)
```

---

### 2. Cherry-pick（挑选提交）

**场景：** 只想要某个分支的特定提交

```bash
# 挑选单个提交
git cherry-pick <commit-hash>

# 挑选多个提交
git cherry-pick <commit1> <commit2>

# 挑选一个范围（不包含 commit1）
git cherry-pick <commit1>..<commit2>
```

**原理：**
```bash
假设分支状态：
main: A ← B ← C
dev:  A ← B ← D ← E ← F

在 main 分支执行 git cherry-pick E：
1. 计算 E 的修改内容（E 相对于 D 的差异）
2. 将这些修改应用到 main 分支的 C 上
3. 创建新的提交 E'

结果：
main: A ← B ← C ← E'
dev:  A ← B ← D ← E ← F
```

---

### 3. Git Tag（标签）

**作用：** 给重要版本打标签（如 v1.0, v2.0）

```bash
# 创建轻量标签
git tag v1.0

# 创建带注释的标签（推荐）
git tag -a v1.0 -m "版本 1.0 发布"

# 给历史提交打标签
git tag -a v0.9 <commit-hash>

# 查看标签
git tag                               # 列出所有标签
git show v1.0                         # 查看标签详情

# 推送标签
git push origin v1.0                  # 推送单个标签
git push origin --tags                # 推送所有标签

# 删除标签
git tag -d v1.0                       # 删除本地标签
git push origin :refs/tags/v1.0       # 删除远程标签
```

**标签的原理：**
```bash
标签是一个指向特定提交的指针（不可移动）
存储位置：.git/refs/tags/

轻量标签：只是一个指针
带注释的标签：包含标签者、日期、注释信息的对象
```

---

### 4. Git Reflog（引用日志）

**作用：** 查看所有操作历史，可以找回"丢失"的提交

```bash
# 查看操作历史
git reflog

# 输出示例：
# abc1234 HEAD@{0}: commit: 添加登录功能
# def5678 HEAD@{1}: reset: moving to HEAD~1
# abc1234 HEAD@{2}: commit: 添加登录功能

# 恢复到某个操作
git reset --hard HEAD@{2}
git reset --hard abc1234
```

**原理：**
```bash
reflog 记录了 HEAD 的所有移动：
- 每次 commit
- 每次 checkout
- 每次 reset
- 每次 merge
- 等等...

保存期限：默认 90 天
```

**救命用法：**
```bash
# 场景：不小心执行了 git reset --hard，想找回
git reflog
# 找到丢失的提交 hash
git reset --hard <hash>
```

---

### 5. 交互式 Rebase

**作用：** 修改历史提交（合并、编辑、删除）

```bash
# 编辑最近 3 个提交
git rebase -i HEAD~3

# 打开编辑器，显示：
# pick abc1234 提交 1
# pick def5678 提交 2
# pick ghi9012 提交 3
```

**可用操作：**
- `pick`：保留提交
- `reword`：修改提交信息
- `edit`：修改提交内容
- `squash`：合并到上一个提交
- `fixup`：合并到上一个提交（丢弃提交信息）
- `drop`：删除提交

**合并提交示例：**
```bash
# 编辑器中改为：
pick abc1234 提交 1
squash def5678 提交 2
squash ghi9012 提交 3

# 保存后会合并成一个提交，并让你编辑合并后的提交信息
```

**⚠️ 注意：** 只能修改未推送的提交！

---

## 第七部分：.gitignore 配置

### 1. 什么是 .gitignore

**作用：** 指定哪些文件不被 Git 跟踪

**常见场景：**
- 依赖目录（node_modules/, vendor/）
- 编译产物（dist/, build/, *.exe）
- 日志文件（*.log）
- 系统文件（.DS_Store）
- 配置文件（.env）

---

### 2. .gitignore 语法

```gitignore
# 注释用 # 开头

# 忽略所有 .log 文件
*.log

# 但不忽略 important.log
!important.log

# 只忽略当前目录的 TODO 文件
/TODO

# 忽略 build 目录下的所有文件
build/

# 忽略 doc 目录下的 .txt 文件
doc/*.txt

# 忽略 doc 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

---

### 3. 常用配置模板

```gitignore
# 依赖目录
node_modules/
vendor/
__pycache__/

# 编译产物
dist/
build/
*.exe
*.o
*.class

# 日志文件
*.log
logs/

# 操作系统文件
.DS_Store
Thumbs.db
desktop.ini

# IDE 配置
.vscode/
.idea/
*.swp
*.swo

# 环境变量
.env
.env.local
.env.production

# 临时文件
tmp/
temp/
*.tmp
```

---

### 4. 已跟踪文件如何忽略

**问题：** 文件已经被 Git 跟踪，加入 .gitignore 后仍然会被提交

**解决方案：**
```bash
# 从 Git 中删除，但保留本地文件
git rm --cached <file>
git rm -r --cached <directory>

# 提交
git commit -m "Remove ignored files"

# 之后 .gitignore 就生效了
```

---

## 第八部分：实战总结

### 1. 日常工作流程

```bash
# ========== 每天开始工作 ==========
git checkout main
git pull origin main

# ========== 开发新功能 ==========
git checkout -b feature/xxx
# ... 编码 ...
git add .
git commit -m "feat: 完成 xxx 功能"

# ========== 合并到主分支 ==========
git checkout main
git pull origin main              # 再次同步
git merge feature/xxx
git push origin main
git branch -d feature/xxx

# ========== 修复 Bug ==========
git checkout -b bugfix/xxx
# ... 修复 ...
git add .
git commit -m "fix: 修复 xxx 问题"
git checkout main
git merge bugfix/xxx
git push origin main
git branch -d bugfix/xxx
```

---

### 2. 关键命令速查表

| 分类 | 命令 | 作用 | 原理 |
|------|------|------|------|
| **初始化** | `git init` | 初始化仓库 | 创建 .git/ 目录 |
| | `git clone <url>` | 克隆仓库 | 复制远程仓库到本地 |
| **查看** | `git status` | 查看状态 | 对比工作区、暂存区、HEAD |
| | `git diff` | 查看差异 | 对比文件内容 |
| | `git log` | 查看历史 | 遍历 commit 链 |
| **基础** | `git add` | 添加到暂存区 | 计算哈希，存储对象 |
| | `git commit` | 提交 | 创建 commit 对象 |
| **分支** | `git branch` | 分支管理 | 创建/删除分支指针 |
| | `git checkout` | 切换分支 | 移动 HEAD 指针 |
| | `git merge` | 合并分支 | 三方合并或快进 |
| **撤销** | `git restore` | 撤销工作区 | 用暂存区覆盖工作区 |
| | `git reset --soft` | 撤销提交 | 移动 HEAD，保留修改 |
| | `git reset --hard` | 完全撤销 | 移动 HEAD，丢弃修改 |
| **远程** | `git push` | 推送 | 上传提交到远程 |
| | `git pull` | 拉取 | fetch + merge |
| | `git fetch` | 获取 | 下载但不合并 |

---

### 3. 三种 reset 模式对比（核心）

```
场景：A ← B ← C ← D (HEAD)
      工作区有修改，暂存区有修改

执行：git reset --soft C

┌─────────┬──────────┬─────────┬─────────┐
│  模式    │ 本地仓库  │ 暂存区   │ 工作区   │
├─────────┼──────────┼─────────┼─────────┤
│ --soft  │ HEAD → C │ 保留修改 │ 保留修改 │
│ --mixed │ HEAD → C │ 清空     │ 保留修改 │
│ --hard  │ HEAD → C │ 清空     │ 清空     │
└─────────┴──────────┴─────────┴─────────┘

记忆口诀：
soft  → 只动仓库（最温柔）
mixed → 动仓库+暂存区（默认）
hard  → 三个全动（最暴力）
```

---

### 4. 冲突解决流程图

```
开始
  ↓
git pull origin main
  ↓
发现冲突？
  ├─ 否 → 完成
  └─ 是 ↓
     git status（查看冲突文件）
     ↓
     手动编辑冲突文件
     ↓
     git add <file>
     ↓
     git commit
     ↓
     git push origin main
     ↓
     完成
```

---

### 5. 分支策略对比

#### Git Flow（复杂项目）
```
master (生产环境)
  └── develop (开发分支)
      ├── feature/* (功能分支)
      ├── release/* (发布分支)
      └── hotfix/* (紧急修复)
```

#### GitHub Flow（推荐）
```
main (主分支，随时可部署)
  └── feature/* (功能分支)
      → Pull Request → Code Review → Merge
```

---

### 6. 提交信息规范

**Conventional Commits：**

```bash
<type>(<scope>): <subject>

# type 类型：
feat:     新功能
fix:      修复 bug
docs:     文档更新
style:    代码格式（不影响功能）
refactor: 重构
test:     测试相关
chore:    构建/工具相关

# 示例：
git commit -m "feat(auth): 添加用户登录功能"
git commit -m "fix(ui): 修复按钮点击无响应"
git commit -m "docs: 更新 README 安装说明"
```

---

### 7. 常见错误和解决方案

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `fatal: not a git repository` | 不在 Git 仓库中 | `cd` 到项目目录或 `git init` |
| `error: failed to push` | 远程有新提交 | `git pull` 再 `git push` |
| `CONFLICT (content)` | 合并冲突 | 手动解决冲突 |
| `refusing to merge unrelated histories` | 两个仓库无共同历史 | `git pull --allow-unrelated-histories` |
| `Your local changes would be overwritten` | 本地有未提交的修改 | `git stash` 或 `git commit` |

---

### 8. Git 工作流程图（总览）

```
┌────────────────────────────────────────────────────────────┐
│                      Git 完整工作流程                        │
└────────────────────────────────────────────────────────────┘

初始化/克隆
    │
    ├─→ git init / git clone
    ↓
配置用户
    │
    ├─→ git config --global user.name/email
    ↓
创建分支
    │
    ├─→ git checkout -b feature/xxx
    ↓
开发功能
    │
    ├─→ 编写代码
    ↓
查看状态
    │
    ├─→ git status / git diff
    ↓
添加到暂存区
    │
    ├─→ git add .
    ↓
提交到本地仓库
    │
    ├─→ git commit -m "message"
    ↓
切换到主分支
    │
    ├─→ git checkout main
    ↓
同步远程
    │
    ├─→ git pull origin main
    ↓
合并功能分支
    │
    ├─→ git merge feature/xxx
    ↓
处理冲突（如果有）
    │
    ├─→ 手动解决 → git add → git commit
    ↓
推送到远程
    │
    ├─→ git push origin main
    ↓
删除功能分支
    │
    └─→ git branch -d feature/xxx
```

---

## 第九部分：核心原理总结

### 1. Git 的底层架构

```
┌─────────────────────────────────────────┐
│            Git 内部结构                   │
├─────────────────────────────────────────┤
│ 工作区 (Working Directory)               │
│   ↓ git add                             │
│ 暂存区 (Staging Area / Index)           │
│   ↓ git commit                          │
│ 本地仓库 (Local Repository)              │
│   ├─ objects/  (对象数据库)              │
│   │   ├─ blob   (文件内容)               │
│   │   ├─ tree   (目录结构)               │
│   │   └─ commit (提交记录)               │
│   ├─ refs/     (引用)                    │
│   │   ├─ heads/   (分支)                 │
│   │   ├─ tags/    (标签)                 │
│   │   └─ remotes/ (远程分支)             │
│   ├─ HEAD      (当前分支指针)            │
│   └─ index     (暂存区索引)              │
│   ↓ git push                            │
│ 远程仓库 (Remote Repository)             │
└─────────────────────────────────────────┘
```

---

### 2. 关键概念总结

| 概念 | 本质 | 存储位置 | 作用 |
|------|------|---------|------|
| **Blob** | 文件内容 | `.git/objects/` | 存储文件数据 |
| **Tree** | 目录结构 | `.git/objects/` | 记录文件名和 Blob 的映射 |
| **Commit** | 提交记录 | `.git/objects/` | 包含 Tree、作者、时间等 |
| **分支** | 指向 Commit 的指针 | `.git/refs/heads/` | 标识开发线 |
| **HEAD** | 指向当前分支的指针 | `.git/HEAD` | 表示当前位置 |
| **暂存区** | 待提交的文件索引 | `.git/index` | 临时存储区 |

---

### 3. 指针关系图

```
HEAD（当前位置）
  ↓
main（分支指针）
  ↓
Commit C（提交对象）
  ├─ tree: Tree T
  ├─ parent: Commit B
  ├─ author: ...
  └─ message: ...
     ↓
Tree T（目录结构）
  ├─ index.html → Blob 1
  ├─ style.css  → Blob 2
  └─ script.js  → Blob 3
```

---

### 4. 核心操作的本质

```bash
# git add 本质
计算文件 SHA-1 → 压缩存储到 objects/ → 更新 index

# git commit 本质
创建 Tree 对象 → 创建 Commit 对象 → 移动分支指针

# git checkout 本质
移动 HEAD 指针 → 更新暂存区 → 更新工作区

# git merge 本质
找共同祖先 → 三方对比 → 创建合并提交

# git reset 本质
移动 HEAD 和分支指针 → 根据模式更新暂存区和工作区

# git push 本质
打包本地新提交 → 发送到远程 → 更新远程分支指针
```

---

## 第十部分：学习路线和资源

### 1. 学习路线

```
入门阶段（1-2 周）
  ├─ 理解 Git 原理（工作区、暂存区、仓库）
  ├─ 掌握基础命令（add, commit, push, pull）
  └─ 学会查看状态和历史

进阶阶段（2-4 周）
  ├─ 分支管理（创建、合并、删除）
  ├─ 撤销操作（restore, reset, revert）
  └─ 冲突解决

高级阶段（1-2 月）
  ├─ rebase 和 cherry-pick
  ├─ stash 和 reflog
  └─ 交互式 rebase

实战应用（持续）
  ├─ 参与团队项目
  ├─ 贡献开源项目
  └─ 建立自己的工作流程
```

---

### 2. 推荐资源

**官方文档：**
- 📖 [Pro Git 电子书（中文）](https://git-scm.com/book/zh/v2)
- 📖 [Git 官方文档](https://git-scm.com/doc)

**可视化学习：**
- 🎮 [Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN)
- 📺 [Git 原理可视化](https://git-school.github.io/visualizing-git/)

**实用工具：**
- 🔧 GitKraken / SourceTree（图形化界面）
- 🔧 VS Code + GitLens 插件
- 🔧 GitHub Desktop

---

### 3. 实践建议

✅ **多练习**
- 在个人项目中使用 Git
- 模拟多人协作场景（创建多个文件夹）
- 故意制造冲突并解决

✅ **参与开源**
- Fork 项目 → 修改 → 提交 Pull Request
- 阅读优秀项目的 Git 历史
- 学习他人的提交信息规范

✅ **建立习惯**
- 频繁小步提交
- 写清晰的提交信息
- 推送前先拉取

✅ **不要害怕**
- Git 几乎可以恢复一切（reflog）
- 最坏情况：重新克隆仓库
- 在测试仓库中大胆尝试

---

## 总结

### Git 的核心思想

1. **快照而非差异**：每次提交保存完整状态
2. **内容寻址**：通过哈希值管理内容
3. **分支便宜**：分支只是指针
4. **分布式**：每个人都有完整历史

### 最重要的命令

```bash
git status                    # 最常用，查看状态
git add .                     # 添加所有修改
git commit -m "message"       # 提交
git push                      # 推送
git pull                      # 拉取
git checkout -b <branch>      # 创建分支
git merge <branch>            # 合并分支
git log --oneline --graph     # 查看历史
```

### 最重要的概念

- **四个工作区**：工作区、暂存区、本地仓库、远程仓库
- **三种 reset**：--soft（温柔）、--mixed（默认）、--hard（暴力）
- **两种合并**：Fast-forward（快进）、Three-way（三方）

---

**最后的建议：**

> Git 是工具，不是障碍
> 
> 理解原理比记住命令更重要
> 
> 从基础开始，循序渐进
> 
> 在实践中成长，在错误中学习
> 
> Keep coding, keep learning! 🚀

---

