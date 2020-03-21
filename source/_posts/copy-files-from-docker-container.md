---
title: 不通过数据卷在docker容器与宿主机间交换数据
tags: ["docker"]
categories: ["tools"]
reward: true
copyright: true
date: 2020-03-21 13:36:03
thumbnail:
---



通常情况下，我们让docker容器与宿主机进行文件传输，最好的方式就是使用挂载数据卷，但偶尔会遇到一些特殊的情况，容器没有挂在数据卷，但容器内又保存着重要的数据，我们希望将它移到宿主机上，但关闭容器重新挂载数据集又会丢失这些数据，为此我们需要一个方法让宿主机和已经启动的容器间能够交换数据。

本文介绍docker的 `cp` 复制指令。

<!--more-->

## 将本地文件复制到docker容器内

```shell
$ sudo docker cp <path-in-localhost> <container-id>:<path-in-container>
```

## 将docker容器内的文件复制到本地

```shell
$ sudo docker cp <container-id>:<path-in-container> <path-in-localhost>
```

