---
layout: post
title: git-commond
tags:
- linux
- git
---

<!--*-->
<!--more-->

## 初始化仓库

```bash
#进入要做成git仓库的文件夹
git init
#这一步初始化git仓库，并生成一个.git的隐藏文件夹
```

## 各种状态

- untracked，未被仓库收纳
- unmodfied，被仓库收纳，提交以后未作修改
- modified， 被仓库收纳，与上一次提交修改过的
- staged， 被仓库收纳add后准备提交

## 简单流程

```bash
#创建一个文件，例如a.py

# 1. 将其添加到仓库
git add a.py
git add . (添加文件下所有文件)

# 2. 提交 message 可替换成对该次提交的描述
git commit -m "message"

# 3. add并commit
# 这会提交所有“已经处于仓库”的文件，新文件不会被提交
git coomit -am "message"

# 4. 当前所有修改和上个版本的区别
git diff

# 5. 将这次提交归并到上一次
git commit --amend [--no-edit](不修改上一次的评论)
```

## 仓库状态


```bash
git status [-s](单行输出)

#从该版本到最初版本的信息
git log [--oneline](单行) [--graph](图形，用于分支)

#所有版本变化的信息
git reflog 
```

## 版本回滚

```bash
git reset --hard HEAD^^^^
# ^ 的数量代表回滚的版本次数，1个回滚1个版本

git reset --hard 版本号
```

版本号可由
git reflog
git log --oneline 
查看

## 单个文件回滚

```bash
git checkout 版本号 -- filename
# --后面有空格

#若再次修改后提交不会影响到之前任何版本的记录
```

## 分支

```bash
#查看所有分支
git branch 

#新建分支, branchname
git branch branchname

#分支跳转
git checkout branchname

#新建分支并跳转
git checkout -b branchname

#删除分支
git branch -d branchname

#将分支归并到主线，此操作必须将分支转到主线
git merge --no-ff -m "message" branchname
```

分支冲突：

```bash
#正常情况下，主分支在分出新分支后不变，新分支更改后可以线性归并到主分支

#若分出新分支后，主分支也有改变，则归并时会报错，会指出有冲突的文件，所以需要在文件中做出修改
#修改完成以后再次提交，此时完成归并
```

## 暂存

当不想提交且需要更改之前版本代码时可将当前状态暂时保存下来

```bash
#暂存
git stash

#弹出上次暂存
git stash pop
```

## [pretty git-log](https://stackoverflow.com/questions/1057564/pretty-git-branch-graphs)

- 好看的git log显示
在~目录下执行`vi .gitconfig`
找到alias 添加`dog = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C    blue<%an>%Creset' --abbrev-commit --date=relative --all`
这样使用`git dog`就能更好的显示log

## 主要命令

![Git命令.png](https://i.loli.net/2018/08/18/5b780848bd950.png)  
