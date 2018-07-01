---
layout: post
date: 2017-08-27 21:57
status: public
tags: git, rebase, cherry-pick, tips, 修改历史
title: 'Git 进阶'
categories: [Programming]
---
# Git 进阶

现在已经有了无数关于 Git 的教程，然而以入门指导为多。虽然足以在 Git 的世界里生存下来，却总偶尔也会因为意外的问题而不知道如何解决。本文会对稍微进阶的内容进行一些讨论。

## 什么是 Git

Git 不过是一个由提交 (commit) 组成的有向无环图 (DAG)，所有的操作都是在这个图上进行的。图中的每一个节点都有一个 ID 作为表示，也就是一个 Sha1 的哈希。但是这样操作起来并不方便，所以我们会给一些重要的节点起一个好记的名字，比如：

1. Tag
1. Branch

这也就是为什么说 Git 的分支是轻量级的，它不过是指向某个提交的指针罢了。
还有一个特殊的东西叫 HEAD， 他总是指向你当前的 commit。如过指向的不是一个 branch 的话，就会得到一个 detached HEAD 的警告。

从这个角度来看常用的一些命令：

1. `git commit`: 以当前 commit 为基础创建一个新的 commit， 并将 HEAD 和 当前分支的指向它
1. `git checkout master`: 将 HEAD 移动到 master 指向的那个 commit
1. `git checkout -b dev`: 为当前 commit 创建一个新的名为 dev 的 branch 指针
1. `git reset HEAD^1`: 将 HEAD 和 当前 branch 指针指向 HEAD^1 位置 (HEAD 的第一个父节点)

## 从 cherry-pick 与 rebase

有时候我们想要对图进行大的修改, 就需要借助 `cherry-pick` 和 `rebase` 这两个命令。简单来说，`cherry-pick` 是复制，`rebase`是剪切。

比如我们有

```graph

       D -- E (dev)
      /
A -- B -- C (master)

```

`git cherry-pick master` 将 master 分支的修改复制到当前分支来，变为

```graph

       D -- E -- C' (dev)
      /
A -- B -- C (master)

```

以及 `git rebase master` 将 master 分支的修改剪切到当 dev 支来， 变为

```graph

A -- B -- C (master) -- D' -- E' (dev)

```

同时借助与 `-i` 参数， 可以对 commit 进行修改和合并，使得历史纪录更加清晰

## 垃圾回收与错误挽救

正如许多编程语言一样，git 也存在着垃圾回收。对于上面指令所删除的 commit, git 并不会立即去删掉它，除非你手动调用 `git gc` 对它进行清理。

这也就给了我们改正错误的机会。

通过 `git reflog` 命令，可以看到之前 `HEAD` 过的 commits。一旦我们发现命令的结果不是所期望的，就可以立即 `reset` 到正确的版本上去。

## 一些小 tips

1. 使用 `git add -p` 确保正确的添加了更改
1. 使用 `git commit -v` 而不是 `git commit -m`
1. 使用 `git commit --amend --no-edit` 快速修改一个 commit
1. 如果不确定效果，试试先加上 `--dry-run` （不是所有命令都支持）
1. 永远不要修改公共的历史
