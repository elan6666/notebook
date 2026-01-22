# Git 使用笔记（从克隆仓库开始｜Obsidian 版）

> 目标：理解 **每一条 Git 命令在干什么、改了哪里、什么时候用**  
> 而不是死记命令。

---

## 一、Git 的核心心智模型（必须先懂）

Git 永远在操作 **4 个区域**：

工作区（Working Directory）  
↓ git add  
暂存区（Staging Area）  
↓ git commit  
本地仓库（Local Repository）  
↓ git push  
远程仓库（GitHub）

- **工作区**：你正在编辑的文件
- **暂存区**：这一次“准备提交”的文件快照
- **本地仓库**：已经提交、不可变的历史
- **远程仓库**：GitHub 上的仓库

---

## 二、从 GitHub 克隆仓库

### 1. 克隆仓库


git clone git@github.com:OWNER/REPO.git



**它在干什么：**

- 下载整个仓库（不是只下载当前代码）
    
- 包含所有历史提交
    
- 在本地创建一个 Git 仓库
    
- 自动设置远程仓库名为 `origin`
    

📌 clone ≠ 下载  
📌 clone = 下载 + 建立版本控制系统

---

### 2. 进入仓库目录

`cd REPO`

Git 命令 **只对当前仓库生效**。

---

### 3. 查看当前状态（最常用）

`git status`

**它会告诉你：**

- 当前在哪个分支
    
- 哪些文件被修改了
    
- 哪些已加入暂存区
    
- 是否有未推送 / 未拉取的提交
    

👉 不确定现在发生了什么，先敲 `git status`

---

## 三、远程仓库相关

### 4. 查看远程仓库地址

`git remote -v`

**作用：**

- 查看远程仓库别名（通常是 `origin`）
    
- 查看对应的 GitHub 地址
    

---

## 四、同步远程更新

### 5. 拉取最新代码

`git pull`

**等价于：**

`git fetch git merge origin/main`

**作用：**

- 下载远程更新
    
- 合并到当前分支
    

---

### 6. 只下载更新，不合并

`git fetch`

**作用：**

- 只更新“远程分支信息”
    
- 不修改你的代码
    
- 非常安全
    

---

## 五、分支（最重要的安全机制）

### 7. 查看本地分支

`git branch`

---

### 8. 创建并切换新分支

`git checkout -b feature/task-x`

**它在干什么：**

- 创建一个分支
    
- 名字叫 `feature/task-x`
    
- 并立刻切换过去
    

📌 `feature/task-x` 是 **一个完整的分支名**  
📌 `/` 只是命名习惯，不是父子分支

---

### 9. 切换分支（新推荐）

`git switch feature/task-x`

只负责切换分支，更安全、语义更清晰。

---

## 六、查看代码改动

### 10. 查看未暂存的改动

`git diff`

**对比：**

- 工作区 vs 暂存区
    

---

### 11. 查看已暂存的改动

`git diff --staged`

**对比：**

- 暂存区 vs 上一次提交
    

---

## 七、提交（三步核心流程）

### 12. 添加文件到暂存区

`git add file.py`

或一次性添加所有：

`git add -A`

**作用：**

- 决定“这一次提交包含哪些改动”
    

---

### 13. 提交代码

`git commit -m "feat: add new module"`

**作用：**

- 将暂存区内容保存为一次提交
    
- 生成 commit ID
    
- 提交内容不可再变
    

📌 commit = 给代码拍一张永久快照

---

### 14. 修改最近一次提交（未 push 时）

`git commit --amend`

**作用：**

- 修改最近一次提交的内容或说明
    

⚠️ 不要在已 push 后使用

---

## 八、推送到 GitHub

### 15. 推送提交

`git push`

**作用：**

- 把本地提交发送到远程仓库
    

---

### 16. 首次推送新分支

`git push -u origin feature/task-x`

**作用：**

- 推送分支
    
- 设置 upstream
    
- 之后只需 `git push`
    

---

## 九、合并与变基

### 17. 合并分支（merge）

`git merge origin/main`

**特点：**

- 保留真实历史
    
- 会产生 merge commit
    

---

### 18. 变基（rebase）

`git rebase origin/main`

**作用：**

- 把当前分支的提交接到最新 main 后
    
- 提交历史更干净
    

⚠️ 不要对已 push、他人使用的分支 rebase

---

## 十、撤销与回滚（救命）

### 19. 撤销工作区改动

`git restore file.py`

---

### 20. 撤销暂存区文件

`git restore --staged file.py`

---

### 21. 安全回滚提交

`git revert <commit_id>`

**特点：**

- 生成反向提交
    
- 不破坏历史
    

---

### 22. 强制回退（危险）

`git reset --hard <commit_id>`

⚠️ 会丢失提交，仅在非常确定时使用

---

## 十一、日常最短流程（肌肉记忆）

`git status git diff git add -A git commit -m "message" git push`