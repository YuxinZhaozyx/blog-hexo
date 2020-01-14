---
title: Ubuntu上从CUDA开始构建深度学习镜像
tags: ["docker", 'pytorch']
categories: ["linux"]
reward: true
copyright: true
date: 2020-01-14 15:53:47
thumbnail: create-deep-learning-docker-image-in-ubuntu/nvidia-docker-logo.png
---





本文介绍如何在ubuntu上从CUDA镜像开始构建深度学习镜像。

<!--more-->

# 前置条件 

+ 已安装 docker 和 nvidia-docker
+ 安装好 nvidia driver （不要求安装 cuda 和 cudnn）

# 构建

## 下载 cuda 镜像

```shell
sudo docker pull nvidia/cuda:9.2-cudnn7-devel
```

更多cuda版本见[此处](https://hub.docker.com/r/nvidia/cuda/tags)。

**注意：** 

+ 下载cuda版本前请先确认宿主机的 nvidia driver 版本高于要安装的 cuda 版本要求，对应关系见[此处](https://github.com/NVIDIA/nvidia-docker/wiki/CUDA)。

+ nvidia driver的版本可以通过 `nvidia-smi` 指令查看。

下载完成后使用以下命令验证是否安装成功：

```shell
sudo nvidia-docker run --rm nvidia/cuda:9.2-cudnn7-devel nvidia-smi
sudo nvidia-docker run --rm nvidia/cuda:9.2-cudnn7-devel nvcc --version
```

若nvidia driver版本不满足要求，下载完成后运行会出现以下错误：

```
docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "process_linux.go:430: container init caused \"process_linux.go:413: running prestart hook 1 caused \\\"error running hook: exit status 1, stdout: , stderr: exec command: [/usr/bin/nvidia-container-cli --load-kmods configure --ldconfig=@/sbin/ldconfig.real --device=all --compute --utility --require=cuda>=10.1 brand=tesla,driver>=384,driver<385 brand=tesla,driver>=396,driver<397 brand=tesla,driver>=410,driver<411 --pid=7808 /home/data/overlay2/5404e537b8e59d7717372339aa3c739b7dc498fe8dc00ca16b69149edab4a498/merged]\\\\nnvidia-container-cli: requirement error: unsatisfied condition: brand = tesla\\\\n\\\"\"": unknown.
```



## 启动cuda容器

```shell
sudo nvidia-docker run --name container_name -it nvidia/cuda:9.2-cudnn7-devel /bin/bash
```

**注：** 

+ `container_name` 可以自己改。

+ 如果中途退出该容器了，可以通过以下指令进入：

  ```shell
  sudo nvidia-docker start container_name
  sudo nvidia-docker attach container_name
  ```

  



## apt-get 换源：

操作指南见[此处](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)。

```shell
apt-get update        # 刷新原本的软件源
apt-get install vim   # 安装 vim
# 按照操作指南更换 apt-get 软件源，可以使用 vi 命令编辑 /etc/apt/sources.list
apt-get update        # 更新为清华镜像源
apt-get install wget  # 安装 wget
```

**注：**必须先 `apt-get update` 否则软件源不会更新，会出现以下错误：

```
E: Unable to locate package vim
```





## 安装 anaconda3

**注：**也可以选择更小的版本[miniconda](https://docs.conda.io/en/latest/miniconda.html)。

在[anaconda官网](https://www.anaconda.com/distribution/#linux)复制安装脚本的链接地址，使用 `wget` 下载：

```shell
cd /home
wget https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh
```

下载后运行脚本即可安装：

```shell
chmod +x Anaconda3-2019.10-Linux-x86_64.sh
./Anaconda3-2019.10-Linux-x86_64.sh
```

安装完成后退出，使安装生效：

```shell
exit
```

重新进入：

```shell
sudo nvidia-docker start container_name
sudo nvidia-docker attach container_name
```

验证是否成功安装：

```shell
conda --version
```



## pip 换源

操作指南见[此处](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)。



## conda 换源

```shell
vi ~/.condarc   # 没有就新建
```

添加以下内容

```yaml
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - defaults
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch
show_channel_urls: true
```



## 设置jupyter notebook

**注：** 由于anaconda自带jupyter，此处略去jupyter的安装步骤。

设置密码：

```shell
jupyter notebook --generate-config
jupyter notebook password
```

后台启动notebook：

```shell
jupyter notebook /base_dir --ip=0.0.0.0 --port=8888 --allow-root &
```

**注：** 使用 CTRL + P + Q 可以退出容器但不关闭容器，保持jupyter的运行



## 安装pytorch

```shell
conda install pytorch torchvision cudatoolkit=9.2  # 不需要加 -c pytorch
```



## 打包成镜像

先退出容器，执行以下命令打包镜像

```shell
sudo docker commit -m="comment" -a="author_name" container_id image_name:tag
```



# 使用

```shell
sudo nvidia-docker run -p 9888:8888 -v /src_dir:/dst_dir --name container_name -it image_name:tag /bin/bash
```

使用中可以用 CTRL + P + Q 退出容器但不关闭容器。

