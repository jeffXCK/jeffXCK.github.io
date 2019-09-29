---
title: git分支相关命令
comments: false
date: 2019-09-29 09:56:22
categories: git
tags: git
---



### 创建分支：
- git branch <分支名>
- git branch -v 查看分支

### 切换分支：
- git checkout <分支名>
- 一步完成： git checkout -b <分支名>

### 合并分支：
- 先切换到主干 git checkout master
- git merge <分支名>

### 删除分支
- 先切换到主干 git checkout master
- git branch -D <分支名>