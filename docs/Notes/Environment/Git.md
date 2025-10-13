# Git 核心原理与实战笔记

> 简洁、清晰、系统的 Git 学习指南
>
> 作者：Claude | 更新日期：2025-10-13

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

#### 3.3 解决冲突的完整