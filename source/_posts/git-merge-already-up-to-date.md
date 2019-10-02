---
title: git merge命令提示Already up to date
tags: ["git", "error"]
categories: ["tools"]
reward: true
copyright: true
date: 2019-07-09 14:45:33
thumbnail:
---





本文描述 `git merge <branch>` 提示 Already up to date 的错误原因及解决方法。

<!--more-->

假定我有两个分支`master`和`dev`。当前处在`dev`分支。

```shell
$ git merge master
Already up to date.
```



## Cause of Errors 错误原因

`Already up to date`意味着已经是最新的了，即`dev`是在`master`的基础上修改的，而`master`自分出`dev`分支后就没修改过，因此`dev`分支是最新的，不需要于`master`合并，因为合并完的仓库和当前的`dev`分支是一模一样的。

其git分支图如下：

```shell
                 E -- F -- G(dev)
                /
A -- B -- C -- D(master)
```

其等同于:

```shell
A -- B -- C -- D(master) -- E -- F -- G(dev)
```

我们想要做的合并实际上是将`master`指向`dev`而已。

## Solution 解决方法

多提交一个空的`commit`，使两条分支叉开，再合并。

```shell
                 E -- F -- G(dev)
                /
A -- B -- C -- D -- H(master)
```

```shell
                 E -- F -- G(dev)
                /           \
A -- B -- C -- D -- H ------ I(master)
```

上图中`H`是一个不做任何修改的提交，其作用是将`master`和`dev`分到两个叉开的分支上，使得`master`不再是`dev`的父节点。

**具体命令:**

```sh
# 移动到master分支下
$ git checkout master
# 在master提交一个空内容
$ git commit --allow-empty -m "ready for merging"
# 合并master和dev分支
$ git merge dev
# 移除dev指针(可选)
$ git branch --delete dev
```