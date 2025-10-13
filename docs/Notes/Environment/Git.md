# Git 完整开发流程演示

## 🎯 项目场景

开发一个简单的待办事项（Todo）应用，模拟真实的团队协作开发流程。

------

## 第一阶段：项目初始化

### 1. 创建项目并初始化 Git

```bash
# 创建项目文件夹
mkdir todo-app
cd todo-app

# 初始化 Git 仓库
git init

# 查看状态
git status
# 输出：On branch main (或 master)
#      No commits yet
```

**发生了什么？**

- 创建了 `.git` 文件夹（本地仓库）
- 当前在 main 分支
- 还没有任何提交记录

------

### 2. 创建初始文件

```bash
# 创建项目文件
echo "# Todo App" > README.md
echo "<!DOCTYPE html>
<html>
<head>
    <title>Todo App</title>
</head>
<body>
    <h1>My Todo List</h1>
</body>
</html>" > index.html

echo "body {
    font-family: Arial;
}" > style.css

# 查看工作区状态
git status
```

**输出：**

```
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md
        index.html
        style.css

nothing added to commit but untracked files present
```

**解读：**

- 文件在 **工作区**
- 状态是 `Untracked`（未跟踪），Git 还不认识这些文件

------

### 3. 添加到暂存区

```bash
# 添加所有文件到暂存区
git add .

# 再次查看状态
git status
```

**输出：**

```
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README.md
        new file:   index.html
        new file:   style.css
```

**解读：**

- 文件已在 **暂存区**
- 状态变成 `Changes to be committed`（准备提交）
- 还没进入本地仓库

------

### 4. 第一次提交

```bash
# 提交到本地仓库
git commit -m "初始化项目：添加基础文件"

# 查看提交历史
git log --oneline
```

**输出：**

```
a1b2c3d (HEAD -> main) 初始化项目：添加基础文件
```

**解读：**

- 文件已保存到 **本地仓库**
- `a1b2c3d` 是提交的哈希值（每次提交的唯一 ID）
- `HEAD -> main` 表示当前在 main 分支

------

## 第二阶段：开发新功能（分支操作）

### 5. 创建功能分支

```bash
# 创建并切换到新分支
git checkout -b feature/add-todo

# 或者用新命令（Git 2.23+）
git switch -c feature/add-todo

# 查看所有分支
git branch
```

**输出：**

```
* feature/add-todo
  main
```

**解读：**

- 创建了新分支 `feature/add-todo`
- `*` 表示当前在这个分支
- 此时工作区的文件和 main 分支一模一样

------

### 6. 在新分支开发功能

```bash
# 添加 JavaScript 文件
echo "function addTodo() {
    const input = document.getElementById('todo-input');
    const list = document.getElementById('todo-list');
    const li = document.createElement('li');
    li.textContent = input.value;
    list.appendChild(li);
    input.value = '';
}" > script.js

# 修改 index.html（添加输入框和按钮）
echo "<!DOCTYPE html>
<html>
<head>
    <title>Todo App</title>
    <link rel='stylesheet' href='style.css'>
</head>
<body>
    <h1>My Todo List</h1>
    <input id='todo-input' type='text' placeholder='输入待办事项'>
    <button onclick='addTodo()'>添加</button>
    <ul id='todo-list'></ul>
    <script src='script.js'></script>
</body>
</html>" > index.html

# 查看状态
git status
```

**输出：**

```
On branch feature/add-todo
Changes not staged for commit:
        modified:   index.html

Untracked files:
        script.js
```

**解读：**

- `index.html` 被修改（状态：modified）
- `script.js` 是新文件（状态：untracked）
- 两者都在 **工作区**，还没进暂存区

------

### 7. 查看修改内容

```bash
# 查看工作区和暂存区的差异
git diff index.html
```

**输出示例：**

```diff
--- a/index.html
+++ b/index.html
@@ -2,8 +2,12 @@
 <html>
 <head>
     <title>Todo App</title>
+    <link rel='stylesheet' href='style.css'>
 </head>
 <body>
     <h1>My Todo List</h1>
+    <input id='todo-input' type='text' placeholder='输入待办事项'>
+    <button onclick='addTodo()'>添加</button>
+    <ul id='todo-list'></ul>
+    <script src='script.js'></script>
 </body>
```

**解读：**

- `-` 表示删除的行
- `+` 表示新增的行
- 可以清楚看到改了什么

------

### 8. 分步提交（精细化提交）

```bash
# 第一步：只提交 script.js
git add script.js
git commit -m "添加 Todo 功能的 JavaScript 逻辑"

# 第二步：提交 index.html
git add index.html
git commit -m "更新 HTML 页面，添加输入框和列表"

# 查看提交历史
git log --oneline --graph
```

**输出：**

```
* d4e5f6g (HEAD -> feature/add-todo) 更新 HTML 页面，添加输入框和列表
* c3d4e5f 添加 Todo 功能的 JavaScript 逻辑
* a1b2c3d (main) 初始化项目：添加基础文件
```

**解读：**

- 功能分支比 main 分支多了 2 个提交
- 提交历史清晰，每个提交做一件事

------

## 第三阶段：合并分支

### 9. 切回主分支并合并

```bash
# 切换到 main 分支
git checkout main

# 查看文件（会发现刚才的修改"消失"了）
cat index.html
# 输出的是旧版本，没有输入框和按钮

# 合并 feature 分支
git merge feature/add-todo

# 查看合并后的状态
git log --oneline --graph --all
```

**输出：**

```
* d4e5f6g (HEAD -> main, feature/add-todo) 更新 HTML 页面，添加输入框和列表
* c3d4e5f 添加 Todo 功能的 JavaScript 逻辑
* a1b2c3d 初始化项目：添加基础文件
```

**解读：**

- 这是 **快进合并**（Fast-forward）
- main 分支直接指向 feature 分支的最新提交
- 现在两个分支指向同一个提交

------

### 10. 删除功能分支

```bash
# 删除已合并的分支
git branch -d feature/add-todo

# 查看分支
git branch
```

**输出：**

```
* main
```

------

## 第四阶段：修改和撤销操作

### 11. 修改文件但想撤销

```bash
# 修改 CSS 文件
echo "body { background: red; }" > style.css

# 糟糕！改错了，想撤销
git status
```

**输出：**

```
Changes not staged for commit:
        modified:   style.css
```

**撤销工作区的修改：**

```bash
# 方法 1（新命令）
git restore style.css

# 方法 2（旧命令）
git checkout -- style.css

# 查看文件，已恢复
cat style.css
# 输出：body { font-family: Arial; }
```

------

### 12. 已经 add 但想撤销

```bash
# 修改文件并添加到暂存区
echo "body { background: blue; }" > style.css
git add style.css

# 查看状态
git status
```

**输出：**

```
Changes to be committed:
        modified:   style.css
```

**从暂存区撤回（但保留工作区的修改）：**

```bash
# 方法 1（新命令）
git restore --staged style.css

# 方法 2（旧命令）
git reset HEAD style.css

# 查看状态
git status
# 输出：Changes not staged for commit
```

**如果还想撤销工作区的修改：**

```bash
git restore style.css
```

------

### 13. 已经 commit 但想撤销

```bash
# 提交了一个错误的修改
echo "body { background: yellow; }" > style.css
git add style.css
git commit -m "修改背景色为黄色"

# 查看历史
git log --oneline
```

**撤销方式 1：回退版本（保留工作区修改）**

```bash
# 回退到上一个版本
git reset --soft HEAD~1

# 查看状态
git status
# 输出：Changes to be committed: modified: style.css
# 撤销了提交，但修改还在暂存区
```

**撤销方式 2：回退版本（清空所有修改）**

```bash
# 完全回退
git reset --hard HEAD~1

# 文件完全恢复到上一个版本
cat style.css
```

**撤销方式 3：创建一个反向提交（推荐用于已推送的提交）**

```bash
# 创建一个新提交来撤销之前的修改
git revert HEAD

# 这会打开编辑器，写提交信息
# 保存后会创建一个新的提交，撤销上一次的修改
```

------

## 第五阶段：远程仓库协作

### 14. 添加远程仓库

```bash
# 在 GitHub/GitLab 上创建仓库后，添加远程地址
git remote add origin https://github.com/username/todo-app.git

# 查看远程仓库
git remote -v
```

**输出：**

```
origin  https://github.com/username/todo-app.git (fetch)
origin  https://github.com/username/todo-app.git (push)
```

------

### 15. 推送到远程仓库

```bash
# 第一次推送（建立追踪关系）
git push -u origin main

# 之后推送只需要
git push
```

**解读：**

- `-u` 参数建立本地 main 和远程 origin/main 的追踪关系
- 推送后，远程仓库就有了本地的所有提交

------

### 16. 拉取远程更新（模拟团队协作）

假设你的同事在远程仓库提交了新代码：

```bash
# 拉取远程更新
git pull origin main

# 或者分两步
git fetch origin        # 只下载，不合并
git merge origin/main   # 手动合并
```

------

### 17. 解决冲突（重要！）

**场景：你和同事同时修改了同一个文件**

```bash
# 你修改了 index.html
echo "修改 A" >> index.html
git add index.html
git commit -m "修改 A"

# 你的同事也修改了同一位置并推送到远程
# 你尝试推送
git push
```

**输出：**

```
! [rejected] main -> main (fetch first)
error: failed to push some refs
```

**解决步骤：**

```bash
# 1. 拉取远程代码
git pull origin main

# 输出：
# Auto-merging index.html
# CONFLICT (content): Merge conflict in index.html
# Automatic merge failed; fix conflicts and then commit the result.

# 2. 查看冲突文件
cat index.html
```

**冲突内容示例：**

```html
<body>
    <h1>My Todo List</h1>
<<<<<<< HEAD
    修改 A
=======
    修改 B（同事的修改）
>>>>>>> origin/main
</body>
```

**3. 手动解决冲突**

```bash
# 编辑文件，选择保留哪个修改（或合并两者）
vim index.html
# 删除冲突标记，保留正确的内容

# 4. 标记冲突已解决
git add index.html

# 5. 完成合并
git commit -m "解决合并冲突"

# 6. 推送
git push
```

------

## 第六阶段：高级操作

### 18. 使用 stash 临时保存工作

```bash
# 你正在开发功能，突然要修复紧急 bug
echo "未完成的功能" >> script.js

# 临时保存当前工作
git stash

# 工作区变干净了，可以切换分支修 bug
git checkout -b hotfix/urgent-bug
# ... 修复 bug 并提交 ...

# 切回原来的分支
git checkout main

# 恢复之前的工作
git stash pop
```

------

### 19. 查看某个文件的修改历史

```bash
# 查看文件的提交历史
git log -- index.html

# 查看每次提交的具体修改
git log -p -- index.html

# 查看谁修改了文件的每一行
git blame index.html
```

------

### 20. 打标签（版本发布）

```bash
# 创建标签
git tag -a v1.0 -m "第一个正式版本"

# 查看所有标签
git tag

# 推送标签到远程
git push origin v1.0

# 推送所有标签
git push origin --tags
```

------

## 📋 完整流程总结

```bash
# 1. 初始化
git init

# 2. 开发流程（循环）
git checkout -b feature/xxx    # 创建功能分支
# ... 编码 ...
git status                     # 查看修改
git diff                       # 查看详细修改
git add .                      # 添加到暂存区
git commit -m "提交信息"       # 提交到本地仓库

# 3. 合并到主分支
git checkout main
git merge feature/xxx
git branch -d feature/xxx

# 4. 同步远程仓库
git pull origin main           # 拉取更新
git push origin main           # 推送到远程

# 5. 撤销操作（按需）
git restore <file>             # 撤销工作区
git restore --staged <file>    # 撤销暂存区
git reset --soft HEAD~1        # 撤销提交（保留修改）
git reset --hard HEAD~1        # 撤销提交（丢弃修改）
```

------

## 🎯 最佳实践

1. **提交频率**：小步提交，每个提交只做一件事

2. **提交信息**：清晰描述做了什么（不要写"修改"、"更新"等模糊词）

3. 分支命名

   ：

   - `feature/功能名` - 新功能
   - `bugfix/bug名` - bug 修复
   - `hotfix/紧急修复` - 线上紧急修复

4. **合并前**：先 `git pull`，确保本地是最新的

5. **不要提交**：不要把敏感信息（密码、密钥）提交到 Git

------

## 🚨 常见错误和解决方案

### 问题 1：提交了敏感文件

```bash
# 从 Git 历史中彻底删除文件
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch config/password.txt" \
  --prune-empty --tag-name-filter cat -- --all
```

### 问题 2：不小心在 main 分支开发了

```bash
# 创建新分支（会带着当前的修改）
git checkout -b feature/xxx

# 在新分支提交
git add .
git commit -m "功能完成"

# 切回 main 分支（此时 main 是干净的）
git checkout main
```

### 问题 3：想修改上一次的提交信息

```bash
git commit --amend -m "新的提交信息"
```

### 问题 4：想合并多个提交

```bash
# 交互式 rebase（合并最近 3 个提交）
git rebase -i HEAD~3
# 在编辑器中将除第一个外的 pick 改为 squash
```

------

这就是完整的 Git 工作流程！建议你自己动手操作一遍，印象会更深刻。