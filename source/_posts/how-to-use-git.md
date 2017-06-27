---
title: GIT简明使用指北
date: 2017-06-25 17:20:02
updated: 2017-06-25 16:38:02
categories: 工具
tags: git
---

### 常用的基本操作命令
- 初始化，将目录变为可用git管理的版本库：`git init`
- 新建或修改的文件添加到暂存区：`git add <file>`
- 暂存区的文件提交到版本库：`git commit -m "messages"`
- 本地库的内容推送到远程库：`git push origin <branch_name>`
- 关联远程库用：`git remote add origin <git_repo>`
- 关联后第一次推送使用：`git push -u origin <branch_name>`
- 克隆一个远程库：`git clone <git_repo>`
- 查看本地分支信息：`git branch`
- 查看远程库信息：`git remote -v`
- 抓取远程的提交：`git pull`


### 分支的操作

{% asset_img sequnce.png 理解分支 %}

- 创建分支`git branch <branch_name>`
- 切换分支`git checkout <branch_name>`
- 创建并切换分支`git checkout -b <branch_name>`
- 查看分支`git branch`
- 快速合并指定分支到当前分支`git merge <branch_name>`，这是`Fast Forward`模式，删除分支后会丢失分支信息，合并后看不出曾经做过合并
- 删除分支`git branch -d <branch_name>`
- 删除未被合并过的分支`git branch -D <name>`强行删除。
- 查看当前git状态`git status`
- 查看提交log`git log`。`--oneline`参数简化显示，`--graph`参数可以看到分支合并图
- 普通合并`git merge --no-ff -m "messages" <branch_name>`，会生成新的commit信息，这样从分支历史上可以看出分支信息


### 暂存工作现场的操作
- 缓存工作现场`git stash`
- 查看已有缓存`git stash list`
- 回到工作现场`git pop`
- 回到指定stash`git stash apply statsh@{n}`
- 删除stash的内容`git stash drop`


### 打标签的操作
- 打标签`git tag <tag_name>`，默认为HEAD，也可以指定某个commit_id
- 查看标签`git tag`
- 查看某条标签信息`git show <tag_name>`
- 推送标签到远程`git push origin <tag_name>`
- 删除本地标签`git tag -d <tag_name>`
- 删除远程标签`git push origin :refs/tags/<tagname>`


### 理解工作区和暂存区

1. 文件修改后为`modified`
2. 未添加到暂存区为`not staged`
3. add后为`staged`

本地修改后add到暂存区，使用`git add <file>`加入暂存区
{% asset_img step1.png 从本地到暂存 %}

暂存区commit到开发分支，使用`git commit -m "commit message"`进行提交
{% asset_img step2.png 从暂存到分支 %}


### 各个区的后悔药

1. 如果只是在工作区修改，并未add，那么可以使用`git checkout -- <file>`来丢弃掉操作
2. 如果已经`add`到暂存区，希望还原到工作区可以使用`git reset HEAD <file>`。其中HEAD表示本地指针。
3. 如果已经`commit`到分支上，希望撤销这次提交，可以使用`git revert <commit_id>`来移除某次commit
4. 如果已经`commit`到分支上并且暂存区干净，可以使用`git reset HEAD~或者<commit_id> `命令将本地指针HEAD指向某次commit，将其还原到工作区上。加上`--hard`表示强制移动指针同时销毁这次commit_id之后的所有操作，本地修改也会被销毁，是比较危险的操作。


### 一些可能出现的问题
> 我怎么获取远程的分支信息?

使用`git fetch --all`

> 获取远程分支信息后使用`git branch`怎么看不到？

使用`git branch -a`可以看到所有，使用`git branch -r`可以看到远程

> 怎么切换到远程的某个分支？

使用`git checkout --track origin/<remote_branch>`

> 不小心把本地的node_modules文件上传到了远程，怎么才能删除远程库上的node_modules文件夹？

1. 首先记得把`/node_modules/`加入到`.gitignore`当中
2. `git rm -r --cached node_modules`
3. `git commit -m 'Remove the now ignored directory node_modules'`
4. `git push origin master`