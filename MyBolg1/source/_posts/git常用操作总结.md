---
title: git常用操作总结
date: 2020-10-10 16:08:42
tags: [git, 基础]
categories: git
index_img: /img/git.jpg
banner_img: /img/git.jpg
---

# git常用操作总结

### 前言

最近在和团队进行协同开发时，发现在git这一块出现的问题非常的多，所以我也抽时间将git重新复习了一遍，这篇博客记录学习的笔记方便以后查看。



### 获取Git仓库

- 现有文件目录使用`git init`初始化仓库
- 从远程服务器上克隆`git clone`一个仓库

### 合并其它分支的代码

- git merge
- git rebase

### 撤销合并merge/rebase

- `git reset --hard [commit]` 工作区、暂存区撤回到制定的commit版本
- `git reset --merge [commit]` 或者 `git reset --merge HEAD^` 撤回到合并前的commit

### 修改某次的commit

- `git commit -amend -m 'comment'` 替换上一次的commit
- `git rebase -i HEAD~[数字n]` 进入vim编辑器，并显示最近n条最新的commit记录

### 撤销某次的commit

- `git revert HEAD` 产生新一次commit来抵消上一次提交

- `git revert commit_id`  撤销中间某次commit

- `git revert -m commit_id` 撤销其他分支合过来的commit

- `git revert --no-commit commit1..commit5` 撤销commit1(不包括)至commit5之间的连续commit

  > git revert的参数：
  >
  > --no-edit：执行时不打开默认编辑器，直接使用 Git 自动生成的提交信息。
  >
  > --no-commit：只抵消暂存区和工作区的文件变化，不产生新的提交。

### 丢弃某次commit

- `git reset [last good SHA]` 丢弃掉某个提交之后的所有提交，在提交历史中彻底消失
- `git reset --hard [last good SHA]` `--hard`参数可以让工作区里面的文件也回到以前的状态

### 查看提交历史记录

- git log 时光机，查看refs的HEAD的指向变动

### 查看文件的改动

- `git diff` 对比工作区与暂存区的同文件, 未添加至暂存区的文件改动
- `git diff --staged/cached`查看添加至暂存区所有文件的内容修改
- `git stage` 显示工作目录和暂存区的状态

### 暂存文件的改动

- `git stash` 临时保存和恢复工作区文件的改动

### 撤销文件的改动

- `git checkout -- [filename]`

### 撤销暂存区的文件

- `git rm --cache [filename]`

