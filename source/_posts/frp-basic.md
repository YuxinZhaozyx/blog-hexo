---
title: frp内网穿透基本使用
tags: ['frp']
categories: ['tools']
reward: true
copyright: true
date: 2020-01-17 21:06:04
thumbnail: frp-basic/architecture.png
---





每次回家连不上学校内网，就没法连接实验室的服务器做实验，有什么办法解决呢？内网穿透是解决这一问题的方法之一。

本文介绍 frp 内网穿透工具的基础使用，仅涉及 frp 的 ssh 访问功能，更多功能见[官方仓库](https://github.com/fatedier/frp)。

<!--more-->

## 前提

+ 一台公网服务器（有公网ip）(没有怎么办？在腾讯云或者百度云上买学生服务器)
+ 一台内网服务器（实验室的服务器）

# 下载

从[github仓库的release](https://github.com/fatedier/frp/releases)中下载最新版本解压即可，公网服务器和内网服务器上都要下载。

也可以用指令下载：

```shell
wget https://github.com/fatedier/frp/releases/download/v0.31.1/frp_0.31.1_linux_amd64.tar.gz
tar -zxvf frp_0.31.1_linux_amd64.tar.gz
```

# 基础使用

## 公网服务器设置

修改 `frps.ini` 文件。

```ini frps.ini
[common]
bind_port = 7000
```

启动

```
./frps -c frps.ini
```

若想后台运行该指令，详见文章 [Linux nohup 后台执行指令](/linux-command-nohup)。

## 内网服务器设置

修改 `frpc.ini` 文件

```ini frpc.ini
[common]
server_addr = x.x.x.x  # 你的公网服务器的ip
server_port = 7000     # 对应公网服务器上frps.ini里bind_port的端口

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000     # 之后访问ssh时使用的端口号
```

启动

```shell
./frpc -c frpc.ini
```

## 使用

```shell
ssh -oPort=6000 username@x.x.x.x
```









