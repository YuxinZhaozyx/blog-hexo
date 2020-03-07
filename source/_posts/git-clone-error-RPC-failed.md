---
title: git clone 克隆大项目时发生error RPC failed
tags: ["git", "error"]
categories: ["tools"]
reward: true
copyright: true
date: 2020-03-07 03:47:56
thumbnail: git-clone-error-RPC-failed/logo.jpg
---





本文介绍如何解决 git 在克隆较大项目时容易发生的错误 `error: RPC failed`。

<!--more-->

## 错误内容

```
error: RPC failed; curl 18 transfer closed with 31812496 bytes remaining to read
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

## 解决方法

通常我们从github克隆是采用https的方式，即

```shell
$ git clone https://github.com/<user-name>/<repo-name>.git
```

这种方式克隆时会有一个buffer的上限，超过此上限时就会报错。

通过使用ssh方式进行克隆可以解决这个问题。

1. 在本机上生成一个新的SSH密钥。

   ```shell
   $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   Enter file in which to save the key (~/.ssh/id_rsa): #存储密钥的路径，直接Enter
   Enter passphrase (empty for no passphrase): # 密码，可设定可不设定，设定后每次上传和下载都需要输入密码
   Enter same passphrase again:  # 再输入一次密码
   ```

   到密钥存储的位置可以查看密钥:

   ```shell
   $ cat ~/.ssh/id_rsa.pub
   ssh-rsa 一长串密钥 you_email@example.com
   ```

2. 在github上添加生成的密钥。

   在github的 settings > SSH and GPG keys 页面下添加新的ssh key，密钥内容为`id_rsa.pub`的全部内容，包含开头的`ssh-rsa`和结尾的邮箱地址。

3. 采用ssh方式进行clone。

   ```shell
   $ git clone git@github.com:<user-name>/<repo-name>.git
   ```

   