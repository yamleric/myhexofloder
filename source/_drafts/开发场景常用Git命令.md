title: 开发场景常用Git命令
tags: []
categories: []
date: 2025-10-04 00:02:53
---
Git是当今软件开发领域不可或缺的版本控制系统。无论是个人项目还是团队协作，掌握Git常用命令都是一项基本技能。本文总结了Git在日常使用中最核心的命令，帮助开发者高效地管理代码。

### Git基本工作流程

Git的典型工作流程如下：

1.  **克隆 (Clone)** 或 **初始化 (Init)** 一个Git仓库。
2.  在工作目录中**修改 (Modify)** 文件。
3.  将修改过的文件**暂存 (Stage)** 到暂存区。
4.  **提交 (Commit)** 暂存区中的文件快照到版本历史中。
5.  将本地的提交**推送 (Push)** 到远程仓库，或从远程仓库**拉取 (Pull)** 更新。

### 常用命令分类详解

为了便于理解和记忆，我们将常用Git命令分为以下几类：

-----

#### 1\. 初始化与配置

| 命令 | 功能描述 | 示例 |
| --- | --- | --- |
| `git init` | 在当前目录初始化一个新的Git仓库。 | `git init` |
| `git clone [url]` | 从远程URL克隆一个仓库到本地。 | `git clone https://github.com/user/repo.git` |
| `git config` | 查看和配置Git的相关设置。 | `git config --global user.name "Your Name"`<br>`git config --global user.email "you@example.com"` |

-----

#### 2\. 日常操作：添加、提交与状态查看

| 命令 | 功能描述 | 示例 |
| --- | --- | --- |
| `git add [file]` | 将文件的修改添加到暂存区。 | `git add .` (添加所有修改)<br>`git add README.md` (添加指定文件) |
| `git commit -m "[message]"` | 将暂存区的内容提交到版本历史，并附上提交信息。 | `git commit -m "feat: add user login feature"` |
| `git status` | 显示工作目录和暂存区的状态。 | `git status` |
| `git diff` | 显示工作目录中尚未暂存的更改。 | `git diff`<br>`git diff --staged` (查看已暂存的更改) |
| `git log` | 显示从近到远的提交日志。 | `git log`<br>`git log --oneline --graph` (单行图形化显示) |

-----

#### 3\. 分支管理

分支是Git的强大功能之一，它允许你在不影响主线开发的情况下进行独立的开发工作。

| 命令 | 功能描述 | 示例 |
| --- | --- | --- |
| `git branch` | 列出、创建或删除分支。 | `git branch` (列出本地分支)<br>`git branch feature-x` (创建新分支)<br>`git branch -d feature-x` (删除分支) |
| `git checkout [branch-name]` | 切换到指定分支。 | `git checkout main` |
| `git checkout -b [new-branch]` | 创建并立即切换到一个新分支。 | `git checkout -b feature-y` |
| `git merge [branch-name]` | 将指定分支的历史合并到当前分支。 | `git checkout main`<br>`git merge feature-y` |

-----

#### 4\. 远程仓库操作

与远程仓库的交互是团队协作的核心。

| 命令 | 功能描述 | 示例 |
| --- | --- | --- |
| `git remote -v` | 列出所有配置的远程仓库。 | `git remote -v` |
| `git remote add [name] [url]` | 添加一个新的远程仓库。 | `git remote add origin https://github.com/user/repo.git` |
| `git fetch [remote-name]` | 从远程仓库下载最新的版本历史，但不合并。 | `git fetch origin` |
| `git pull [remote-name] [branch-name]` | 从远程仓库抓取更新，并与本地分支合并。 | `git pull origin main` |
| `git push [remote-name] [branch-name]` | 将本地分支的提交推送到远程仓库。 | `git push origin main` |

-----

#### 5\. 撤销与修改

在开发过程中，难免需要撤销或修改某些操作。

| 命令 | 功能描述 | 示例 |
| --- | --- | --- |
| `git checkout -- [file]` | 丢弃工作目录中对指定文件的修改。 | `git checkout -- README.md` |
| `git reset HEAD [file]` | 将文件从暂存区撤回到工作目录。 | `git reset HEAD README.md` |
| `git reset [commit]` | 重置当前分支的HEAD到指定的提交，有多种模式（`--soft`, `--mixed`, `--hard`）。 | `git reset --hard HEAD^` (回退到上一个版本) |
| `git revert [commit]` | 创建一个新的提交来撤销指定的提交，这是一种更安全的回退方式。 | `git revert abc54321` |
| `git commit --amend` | 修改最后一次的提交。 | `git commit --amend -m "A new commit message"` |
| `git stash` | 临时保存当前未提交的修改，使工作目录变干净。 | `git stash`<br>`git stash pop` (恢复并删除储藏) |

掌握以上这些核心命令，你将能够自信地在日常开发中使用Git进行版本控制，并与团队进行高效协作。建议多加练习，熟能生巧。