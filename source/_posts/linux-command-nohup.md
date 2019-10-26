---
title: Linux nohup 后台执行指令
tags: ["command"]
categories: ["linux"]
reward: true
copyright: true
date: 2019-10-21 00:09:18
thumbnail:
---



通常情况下，执行 Linux 指令必须终端一直开着才能指令执行下去，直到指令执行完成，才会结束。`nohup` 指令可以让指令在后台执行，而不需要操作者一直等待其执行完成，即使操作者登出指令依然会继续执行。

<!--more-->

nohup 全程为 no hangup。 hangup (HUP) 信号会在用户登出时由系统发出，通知所有进程结束，但通过 `nohup` 执行的指令，HUP信号会被 `nohup` 拦截，从而让指令继续执行。

# 后台运行

```shell
nohup your-command &
```

`nohup` 默认会将输出重定向到  nohup.out  文件，注意默认是将输出追加到文件尾。

可以通过 `tail` 指令自动"即时"显示最新的输出。

**注：** 不一定能看到即时的输出，因为 stdout 通常有缓存，只有缓存满了才会写到输出文件中

```shell
tail -f nohup.out
```

也可以自已重定向输出到指定文件。

```shell
nohup your-command > output.file 2>&1 &
```

## 以低优先级执行

因为使用 `nohup` 执行的指令一般都是要跑很久的，用户登出后继续运行，有时候可能占用太多的系统资源，我们可以结合 `nice` 指令让指令以较低优先级在后台执行，尽量不影响其他正常指令的执行。

```shell
nohup nice your-command &
```

# 查看后台进程

由于 `nohup` 指令在后台执行，要查看及终止后台进程都需要先获取该进程的 pid。

```shell
jobs -l
```

**输出样例：** 

```shell
[1]+   490 Running    nohup go run main.go &
```

其中，1 是任务标识 job-spec，490 是进程的 pid。

注： `jobs` 命令在重新登录后就无法查看后台进程了，可以使用 `top` 指令查看所有进程。

```shell
top
```



# 终止后台进程

```shell
kill your-nohup-pid
```

# 将后台程序移到前台

```shell
fg your-nohup-job-spec
```

**注：**虽然移到了前台，但 stdout 仍是带缓存的，只有等缓冲区满了，输出才会刷新到屏幕上。