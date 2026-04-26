---
title: Git 分支操作
type: entity
tags: [full-stack]
sources: []
created: 2026-04-26
updated: 2026-04-26
summary: Git 分支常用命令速查，涵盖查看、创建、切换、删除分支的日常操作。
---

# Git 分支操作

Git 分支常用命令速查，涵盖查看、创建、切换、删除分支的日常操作。

## 常用命令

```bash
git branch -vv                                    # 查看本地分支（含追踪状态）
git branch -r                                     # 查看远程分支
git switch -c feature/xxx                         # 创建并切换分支
git switch feature/xxx                            # 切换已有分支
git switch -c feature/xxx origin/feature/xxx      # 从远程分支创建本地分支
git branch -d feature/xxx                         # 删除本地分支
```

## 紧急切换场景

在 feature-A 分支工作到一半，需要紧急切到 master 修 bug：

```bash
git stash
git switch master
# 修完 bug 后
git switch feature-A
git stash pop
```

## 相关实体

- [[entities/github-reference]]
