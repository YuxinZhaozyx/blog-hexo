---
title: linux上安装docker CE
tags: ['docker']
categories: ['tools']
reward: true
copyright: true
date: 2020-01-17 21:39:02
thumbnail: docker-linux-install/logo.jpg
---



本文介绍在linux系统上安装docker CE，官方手册见[此处](https://docs.docker.com/install/linux/docker-ce/ubuntu)。

<!--more-->

# 设置仓库

**注:** 此"仓库"不是指github仓库，而是apt的软件源。

1. 安装一些必要的库

   ```shell
   sudo apt-get update
   sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg-agent \
       software-properties-common
   ```

2. 添加docker官方的GPG密钥

   ```shell
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

3. 加入docker的仓库

   机器是 x86_64/amd64 架构的，指令如下：（其他架构见[官方手册](https://docs.docker.com/install/linux/docker-ce/ubuntu#set-up-the-repository)）

   ```shell
   sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   ```

# 安装

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

