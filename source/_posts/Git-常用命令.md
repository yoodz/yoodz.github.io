---
title: Git 常用命令
description: Git 常用命令
tags: git
categories: Tools
abbrlink: 5fddf106
date: 2019-06-30 18:57:00
---

```javascript
//在当前目录新建一个Git代码库
git init

// 添加改动文件到暂存区
git add .

//停止追踪指定文件，但该文件会保留在工作区（如果添加ignorance文件无效的话）
git rm --cached [file] 

//提交暂存区到仓库区
git commit -m [message]

// 恢复暂存区的所有文件到工作区
git checkout .

// 暂时将未提交的变化移除，稍后再移入
git stash
git stash pop

//添加远程仓库
git remote add origin git://github.com/paulboone/ticgit.git

//添加.gitignore文件，忽略node_modules
node_modules/
```