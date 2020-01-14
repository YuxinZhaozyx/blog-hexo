---
title: Docker基础
tags: ["docker"]
categories: ["tools"]
reward: true
copyright: true
date: 2019-10-02 18:09:05
thumbnail:
---







本文讲述 docker 的基本概念和使用。 

<!--more-->



## 容器

Docker 利用容器来运行应用。

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

### 用容器执行指令

```shell
docker run ubuntu:15.10 /bin/echo "hello world"
```

### 进行交互式的容器

```shell
docker run -i -t ubuntu:15.10 /bin/bash
root@dc005c79503:/# 
```

+ `-t` : 再新容器内指定一个伪终端或终端
+ `-i` : 允许你对容器内的标准输入进行交互

可以通过运行exit命令或者使用CTRL+D来退出容器

使用CTRL+P+Q可以退出容器但不关闭容器



### 后台启动容器

```shell
docker run -d <image> <command>
```

```shell
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

+ `-d` : 让容器后台运行

输出不会打印 "hello world" 而是容器的ID

### 容器命名

```shell
docker run -d -P --name container_name <image> <command>
```

### 将容器的端口映射到本机的端口上

```shell
docker run -d -P <image> <command>
docker run -d -p 8000:5000 <image> <command>
```

- `-d` : 让容器后台运行
- `-P` : 将容器内部使用的网络端口映射到我们使用的主机上(随机)
- `-p` : 将容器内部使用的端口映射到我们使用 的主机上(将容器内的5000端口映射到本地主机的8000端口上)

```shell
docker run -d -p 192.168.99.100:8000:5000/udp <image> <command> # 指定host和port
```

### 查看所有的容器

```shell
docker ps     # 查看正在允许的容器
docker ps -a  # 查看所有容器
docker ps -l  # 查看最后一次创建的容器
```

### 查看容器输出

```shell
docker logs <name/id>
docker logs -f <name/id>
```

- `-f` : 让 docker logs 在输出后不是直接退出，而是一直接收该容器的输出

### 查看容器内部运行的进程

```shell
docker top <name/id>
```

### 查看容器的配置和状态信息

```shell
docker inspect <name/id>
```

### 查看网络端口

```shell
docker port <name/id>
```

### 停止容器

```docker 
docker stop <name/id>
```

### 重启容器

将已经停止的容器再启动

```shell
docker start <name/id>
```

将一个运行态的容器终止再重新启动启动

```shell
docker restart <name/id>
```

### 移除容器

```shell
docker rm <name/id>
```

:warning: 移除容器时，该容器必须是停止状态，否则会报错

### 在运行的容器中执行命令

```shell
docker exec -d -i -t <container name/id> <command>
```

+ `-d` : 在后台运行
+ `-t` : 分配一个伪终端(pseudo-tty)并绑定到容器的stdin上
+ `-i` : 保持容器的stdin打开

### 进入容器

```shell
docker attach <container>
```

```shell
docker run -d -i -t --name my_python python
docker attach my_python
>>> print("hello")
```

但是使用 `attach` 命令有时候并不方便。当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。

### 导出容器

```shell
docker export -o <output_file_name> <container name/id>
docker export <container name/id> > <output_file_name>
```

```shell
docker export my_python > my_python.tar
```

### 导入容器为镜像

```shell
docker import <input_file> <new container name>
```

```shell
docker import my_python.tar new-python:dev
docker mport http://example.com/exampleimage.tgz example/imagerepo
```

用户既可以使用 `docker load` 来导入镜像存储文件到本地镜像库，也可以使用 `docker import` 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。







## 镜像

Docker 镜像就是一个只读的模板。

Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

### 列出镜像列表

```shell
docker images
```

### 查找镜像

可以再 Docker Hub 网站上查找也可以用命令行查找

```shell
docker search <image>
```

### 拖取镜像

```shell
docker pull <image>
```

### 修改/更新镜像

```shell
docker run -t -i ubuntu:14.01 /bin/bash
root@<container id>:/#  
```

即可使用容器内的shell镜像操作，容器的内容会因此发生变化，需要提交容器的副本

```shell
docker commit -m="has update" -a="my name" <container id> myname/ubuntu:v2 
```

+ `-m` : 提交的描述信息
+ `-a` : 指定镜像作者
+ `<container id>`: 容器ID
+ `myname/ubuntu:v2` : 指定要创建的目标镜像名, 其中 `v2` 为tag

使用这个新的镜像启动容器

```shell
docker run -t -i myname/ubuntu:v2 /bin/bash
```

### Dockerfile 构建镜像

从零开始创建一个新的 Docker 镜像，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

```dockerfile
FROM    centos:6.7
MAINTAINER  my name "my@mail.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd myname
RUN     /bin/echo 'myname:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
ADD		myAPP  /var/www
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```

每一个指令都会在镜像上创建一个新的层，:warning: 一个镜像不能超过127层，每一个指令的前缀都必须是大写的。

+ `FROM` : 指定使用哪个镜像作为基础
+ `RUN` : 指令会在创建中运行，可以用来安装些环境
+ `ADD` : 复制本地文件到镜像（此处的myAPP是本地的一个文件夹）
+ `EXPOSE` : 向外部开放端口
+ `CMD` : 描述容器启动后运行的指令

**构建镜像**

```shell
docker build -t myname/centos:6.7 .
```

+ `-t` : 指定要创建的目标镜像名和tag
+ `.` : Dockerfile文件所在目录，这里是当前目录

### 设置镜像标签

```shell
docker tag <image id> <image name: tag>
```

```shell
docker tag 860c279d2fec myname/centos:dev
docker images
```

```
REPOSITORY       TAG    IMAGE ID        CREATED        SIZE
runoob/centos    6.7    860c279d2fec    5 hours ago    190.6 MB
runoob/centos    dev    860c279d2fec    5 hours ago    190.6 MB
```

### 导出镜像到本地

```shell
docker save -o <output_file_name> <image>
```

```shell
docker save -o python_from_docker.tar python:latest
```

### 从本地载入镜像

```shell
docker load --input <input_file_name>
# 或
docker load < <input_file_name>
```

```shell
docker load --input python_from_docker.tar
docker load < python_from_docker.tar
```

### 移除镜像

```shell
docker rmi <image>
```

```shell
docker rmi python:lastest
```







## 仓库

仓库（Repository）是集中存放镜像的地方。

一个容易混淆的概念是注册服务器（Registry）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址 `dl.dockerpool.com/ubuntu` 来说，`dl.dockerpool.com` 是注册服务器地址，`ubuntu` 是仓库名。

### Docker Hub

目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)，其中已经包括了超过 15,000 的镜像。大部分需求，都可以通过在 Docker Hub 中直接下载镜像来实现。

#### 登录

```shell
docker login
Username: your_docker_hub_user_name
Password: your_docker_hub_password
```

注册成功后，本地用户目录的 `.dockercfg` 中将保存用户的认证信息。

#### 搜索镜像

```shell
docker search <image>
```

#### 拉取镜像

```shell
docker pull <image>
```

#### 推送镜像到Docker Hub

```shell
docker push <image>
```

#### 自动创建

自动创建（Automated Builds）功能对于需要经常升级镜像内程序来说，十分方便。 有时候，用户创建了镜像，安装了某个软件，如果软件发布新版本则需要手动更新镜像。。

而自动创建允许用户通过 Docker Hub 指定跟踪一个目标网站（目前支持 GitHub 或 BitBucket）上的项目，一旦项目发生新的提交，则自动执行创建。

要配置自动创建，包括如下的步骤：

- 创建并登录 Docker Hub，以及目标网站；
- 在目标网站中连接帐户到 Docker Hub；
- 在 Docker Hub 中 [配置一个自动创建](https://registry.hub.docker.com/builds/add/)；
- 选取一个目标网站中的项目（需要含 Dockerfile）和分支；
- 指定 Dockerfile 的位置，并提交创建。

之后，可以 在Docker Hub 的自动创建页面中跟踪每次创建的状态。

### 私有仓库

`docker-registry` 是官方提供的工具，可以用于构建私有的镜像仓库。

#### 容器安装docker-registry

在安装了 Docker 后，可以通过获取官方 registry 镜像来运行。

```shell
docker run -d -p 5000:5000 registry
```

这将使用官方的 registry 镜像来启动本地的私有仓库。 用户可以通过指定参数来配置私有仓库位置，例如配置镜像存储到 Amazon S3 服务。

```shell
docker run \
    -e SETTINGS_FLAVOR=s3 \
    -e AWS_BUCKET=acme-docker \
    -e STORAGE_PATH=/registry \
    -e AWS_KEY=AKIAHSHB43HS3J92MXZ \
    -e AWS_SECRET=xdDowwlK7TJajV1Y7EoOZrmuPEJlHYcNP2k4j49T \
    -e SEARCH_BACKEND=sqlalchemy \
    -p 5000:5000 \
    registry
```

此外，还可以指定本地路径（如 `/home/user/registry-conf` ）下的配置文件。

```shell
docker run -d -p 5000:5000 -v /home/user/registry-conf:/registry-conf -e DOCKER_REGISTRY_CONFIG=/registry-conf/config.yml registry
```

默认情况下，仓库会被创建在容器的 `/tmp/registry` 下。可以通过 `-v` 参数来将镜像文件存放在本地的指定路径。 例如下面的例子将上传的镜像放到 `/opt/data/registry` 目录。

```shell
docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
```

#### 本地安装docker-registry

对于 Ubuntu 或 CentOS 等发行版，可以直接通过源安装。

- Ubuntu

```
$ sudo apt-get install -y build-essential python-dev libevent-dev python-pip liblzma-dev
$ sudo pip install docker-registry
```

- CentOS

```
$ sudo yum install -y python-devel libevent-devel python-pip gcc xz-devel
$ sudo python-pip install docker-registry
```

也可以从 [docker-registry](https://github.com/docker/docker-registry) 项目下载源码进行安装。

```
$ sudo apt-get install build-essential python-dev libevent-dev python-pip libssl-dev liblzma-dev libffi-dev
$ git clone https://github.com/docker/docker-registry.git
$ cd docker-registry
$ sudo python setup.py install
```

然后修改配置文件，主要修改 dev 模板段的 `storage_path` 到本地的存储仓库的路径。

```
$ cp config/config_sample.yml config/config.yml
```

之后启动 Web 服务。

```
$ sudo gunicorn -c contrib/gunicorn.py docker_registry.wsgi:application
```

或者

```
$ sudo gunicorn --access-logfile - --error-logfile - -k gevent -b 0.0.0.0:5000 -w 4 --max-requests 100 docker_registry.wsgi:application
```

此时使用 curl 访问本地的 5000 端口，看到输出 docker-registry 的版本信息说明运行成功。

> 注：`config/config_sample.yml` 文件是示例配置文件。

#### 在私有仓库上传、下载、搜索镜像

创建好私有仓库之后，就可以使用 `docker tag` 来标记一个镜像，然后推送它到仓库，别的机器上就可以下载下来了。例如私有仓库地址为 `192.168.7.26:5000`。

先在本机查看已有的镜像。

```shell
docker images
REPOSITORY      TAG        IMAGE ID         CREATED            VIRTUAL SIZE
ubuntu          latest     ba5877dc9bec     6 weeks ago        192.7 MB
ubuntu          14.04      ba5877dc9bec     6 weeks ago        192.7 MB
```

使用`docker tag` 将 `ba58` 这个镜像标记为 `192.168.7.26:5000/test`（格式为 `docker tag IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]`）。

```shell
docker tag ba58 192.168.7.26:5000/test
root ~ # docker images
REPOSITORY               TAG        IMAGE ID        CREATED        VIRTUAL SIZE
ubuntu                   14.04      ba5877dc9bec    6 weeks ago    192.7 MB
ubuntu                   latest     ba5877dc9bec    6 weeks ago    192.7 MB
192.168.7.26:5000/test   latest     ba5877dc9bec    6 weeks ago    192.7 MB
```

使用 `docker push` 上传标记的镜像。

```shell
docker push 192.168.7.26:5000/test
The push refers to a repository [192.168.7.26:5000/test] (len: 1)
Sending image list
Pushing repository 192.168.7.26:5000/test (1 tags)
Image 511136ea3c5a already pushed, skipping
Image 9bad880da3d2 already pushed, skipping
Image 25f11f5fb0cb already pushed, skipping
Image ebc34468f71d already pushed, skipping
Image 2318d26665ef already pushed, skipping
Image ba5877dc9bec already pushed, skipping
Pushing tag for rev [ba5877dc9bec] on {http://192.168.7.26:5000/v1/repositories/test/tags/latest}
```

用 curl 查看仓库中的镜像。

```
curl http://192.168.7.26:5000/v1/search
{"num_results": 7, "query": "", "results": [{"description": "", "name": "library/miaxis_j2ee"}, {"description": "", "name": "library/tomcat"}, {"description": "", "name": "library/ubuntu"}, {"description": "", "name": "library/ubuntu_office"}, {"description": "", "name": "library/desktop_ubu"}, {"description": "", "name": "dockerfile/ubuntu"}, {"description": "", "name": "library/test"}]}
```

这里可以看到 `{"description": "", "name": "library/test"}`，表明镜像已经被成功上传了。

现在可以到另外一台机器去下载这个镜像。

```
docker pull 192.168.7.26:5000/test
Pulling repository 192.168.7.26:5000/test
ba5877dc9bec: Download complete
511136ea3c5a: Download complete
9bad880da3d2: Download complete
25f11f5fb0cb: Download complete
ebc34468f71d: Download complete
2318d26665ef: Download complete
$ sudo docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
192.168.7.26:5000/test             latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

可以使用 [这个脚本](https://github.com/yeasy/docker_practice/raw/master/_local/push_images.sh) 批量上传本地的镜像到注册服务器中，默认是本地注册服务器 `127.0.0.1:5000`。例如：

```
$ wget https://github.com/yeasy/docker_practice/raw/master/_local/push_images.sh; sudo chmod a+x push_images.sh
$ ./push_images.sh ubuntu:latest centos:centos7
The registry server is 127.0.0.1
Uploading ubuntu:latest...
The push refers to a repository [127.0.0.1:5000/ubuntu] (len: 1)
Sending image list
Pushing repository 127.0.0.1:5000/ubuntu (1 tags)
Image 511136ea3c5a already pushed, skipping
Image bfb8b5a2ad34 already pushed, skipping
Image c1f3bdbd8355 already pushed, skipping
Image 897578f527ae already pushed, skipping
Image 9387bcc9826e already pushed, skipping
Image 809ed259f845 already pushed, skipping
Image 96864a7d2df3 already pushed, skipping
Pushing tag for rev [96864a7d2df3] on {http://127.0.0.1:5000/v1/repositories/ubuntu/tags/latest}
Untagged: 127.0.0.1:5000/ubuntu:latest
Done
Uploading centos:centos7...
The push refers to a repository [127.0.0.1:5000/centos] (len: 1)
Sending image list
Pushing repository 127.0.0.1:5000/centos (1 tags)
Image 511136ea3c5a already pushed, skipping
34e94e67e63a: Image successfully pushed
70214e5d0a90: Image successfully pushed
Pushing tag for rev [70214e5d0a90] on {http://127.0.0.1:5000/v1/repositories/centos/tags/centos7}
Untagged: 127.0.0.1:5000/centos:centos7
Done
```

### 仓库配置文件

Docker 的 Registry 利用配置文件提供了一些仓库的模板（flavor），用户可以直接使用它们来进行开发或生产部署。

#### 模板

在 `config_sample.yml` 文件中，可以看到一些现成的模板段：

- `common`：基础配置
- `local`：存储数据到本地文件系统
- `s3`：存储数据到 AWS S3 中
- `dev`：使用 `local` 模板的基本配置
- `test`：单元测试使用
- `prod`：生产环境配置（基本上跟s3配置类似）
- `gcs`：存储数据到 Google 的云存储
- `swift`：存储数据到 OpenStack Swift 服务
- `glance`：存储数据到 OpenStack Glance 服务，本地文件系统为后备
- `glance-swift`：存储数据到 OpenStack Glance 服务，Swift 为后备
- `elliptics`：存储数据到 Elliptics key/value 存储

用户也可以添加自定义的模版段。

默认情况下使用的模板是 `dev`，要使用某个模板作为默认值，可以添加 `SETTINGS_FLAVOR` 到环境变量中，例如

```shell
export SETTINGS_FLAVOR=dev
```

另外，配置文件中支持从环境变量中加载值，语法格式为 `_env:VARIABLENAME[:DEFAULT]`。

#### 示例配置

```shell
common:
    loglevel: info
    search_backend: "_env:SEARCH_BACKEND:"
    sqlalchemy_index_database:
        "_env:SQLALCHEMY_INDEX_DATABASE:sqlite:////tmp/docker-registry.db"

prod:
    loglevel: warn
    storage: s3
    s3_access_key: _env:AWS_S3_ACCESS_KEY
    s3_secret_key: _env:AWS_S3_SECRET_KEY
    s3_bucket: _env:AWS_S3_BUCKET
    boto_bucket: _env:AWS_S3_BUCKET
    storage_path: /srv/docker
    smtp_host: localhost
    from_addr: docker@myself.com
    to_addr: my@myself.com

dev:
    loglevel: debug
    storage: local
    storage_path: /home/myself/docker

test:
    storage: local
    storage_path: /tmp/tmpdockertmp
```







## 数据卷 /数据管理

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- 数据卷可以在容器之间共享和重用
- 对数据卷的修改会立马生效
- 对数据卷的更新，不会影响镜像
- 卷会一直存在，直到没有容器使用。数据卷的使用，类似于 Linux 下对目录或文件进行 mount

### 创建数据卷

在用 `docker run` 命令的时候，使用 `-v` 标记来创建一个数据卷并挂载到容器里。在一次 run 中多次使用可以挂载多个数据卷。

下面创建一个 web 容器，并加载一个数据卷到容器的 `/webapp` 目录。

```shell
docker run -d -P --name web -v /webapp training/webapp python app.py
```

+ `-v` : 创建数据卷或挂载文件夹/文件到容器中

>  注意：也可以在 Dockerfile 中使用 `VOLUME` 来添加一个或者多个新的卷到由该镜像创建的任意容器。

### 挂在一个主机目录作为数据卷

```shell
docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```

上面的命令加载主机的 `/src/webapp` 目录到 `/opt/webapp` 目录。 这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，如果目录不存在 Docker 会自动为你创建它。

> 注意：Dockerfile 中不支持这种用法，这是因为 Dockerfile 是为了移植和分享用的。然而，不同操作系统的路径格式不一样，所以目前还不能支持。

Docker 挂载数据卷的默认权限是读写，用户也可以通过 `:ro` 指定为只读。

```shell
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro
training/webapp python app.py
```

### 挂载一个本地主机文件作为数据卷

```shell
docker run --rm -i -t -v ~/.bash_history:/.bash_history ubuntu /bin/bash
```

### 数据卷容器 / 容器间共享持续更新的数据

创建一个命名的数据卷容器 dbdata。

```shell
docker run -d -v /dbdata --name dbdata <container> <command>
```

```shell
docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres
```

在其他容器中使用 `--volumes-from` 来挂载 dbdata 容器中的数据卷。

```shell
docker run -d --volumes-from dbdata --name db1 training/postgres
docker run -d --volumes-from dbdata --name db2 training/postgres
```

还可以使用多个 `--volumes-from` 参数来从多个容器挂载多个数据卷。 也可以从其他已经挂载了数据卷的容器来挂载数据卷。

```
docker run -d --name db3 --volumes-from db1 training/postgres
```

:warning: 使用 `--volumes-from` 参数所挂载数据卷的容器自己并不需要保持在运行状态。

:warning: 如果删除了挂载的容器（包括 dbdata、db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用 `docker rm -v` 命令来指定同时删除关联的容器。 这可以让用户在容器之间升级和移动数据卷。

### 数据卷备份

首先使用 `--volumes-from` 标记来创建一个加载 dbdata 容器卷的容器，并从本地主机挂载当前到容器的 /backup 目录。

```shell
docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

容器启动后，使用了 `tar` 命令来将 dbdata 卷备份为本地的 `/backup/backup.tar`。

### 数据卷恢复

如果要恢复数据到一个容器，首先创建一个带有数据卷的容器 dbdata2。

```shell
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

然后创建另一个容器，挂载 dbdata2 的容器，并使用 `untar` 解压备份文件到挂载的容器卷中。

```shell
docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf
/backup/backup.tar
```







## 容器网络

### 容器互联

使用 `--link` 参数可以让容器之间安全的进行交互。

创建一个数据库容器。

```shell
docker run -d --name db training/postgres
```

然后创建一个 web 容器，并将它连接到 db 容器

```shell
docker run -d -P --name web --link db:db training/webapp python app.py
```

此时，db 容器和 web 容器建立互联关系。

`--link` 参数的格式为 `--link name:alias`，其中 `name` 是要链接的容器的名称，`alias` 是这个连接的别名。

使用 `docker ps` 来查看容器的连接

```shell
docker ps
IMAGE                     ...  PORTS                    NAMES
training/postgres:latest  ...  5432/tcp                 db, web/db
training/webapp:latest    ...  0.0.0.0:49154->5000/tcp  web
```

可以看到自定义命名的容器，db 和 web，db 容器的 names 列有 db 也有 web/db。这表示 web 容器链接到 db 容器，web 容器将被允许访问 db 容器的信息。

Docker 在两个互联的容器之间创建了一个安全隧道，而且不用映射它们的端口到宿主主机上。在启动 db 容器的时候并没有使用 `-p` 和 `-P` 标记，从而避免了暴露数据库端口到外部网络上。

Docker 通过 2 种方式为容器公开连接信息：

- 环境变量
- 更新 `/etc/hosts` 文件

使用 `env` 命令来查看 web 容器的环境变量

```shell
docker run --rm --name web2 --link db:db training/webapp env
. . .
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5000_TCP=tcp://172.17.0.5:5432
DB_PORT_5000_TCP_PROTO=tcp
DB_PORT_5000_TCP_PORT=5432
DB_PORT_5000_TCP_ADDR=172.17.0.5
. . .
```

其中 DB_ 开头的环境变量是供 web 容器连接 db 容器使用，前缀采用大写的连接别名。

除了环境变量，Docker 还添加 host 信息到父容器的 `/etc/hosts` 的文件。下面是父容器 web 的 hosts 文件

```shell
docker run -t -i --rm --link db:db training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.5  db
```

这里有 2 个 hosts，第一个是 web 容器，web 容器用 id 作为他的主机名，第二个是 db 容器的 ip 和主机名。 可以在 web 容器中安装 ping 命令来测试跟db容器的连通。

```shell
root@aed84ee21bde:/opt/webapp# apt-get install -yqq inetutils-ping
root@aed84ee21bde:/opt/webapp# ping db
PING db (172.17.0.5): 48 data bytes
56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms
56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms
56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms
```

用 ping 来测试db容器，它会解析成 `172.17.0.5`。





## Dockerfile

### 基本结构

Dockerfile 由一行行命令语句组成，并且支持以 `#` 开头的注释行。

一般的，Dockerfile 分为四部分：

+ 基础镜像信息
+ 维护者信息
+ 镜像操作指令
+ 容器启动时执行指令

```dockerfile
# Nginx
#
# VERSION               0.0.1

FROM      ubuntu
MAINTAINER Victor Vieux <victor@docker.com>

RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server

# Firefox over VNC
#
# VERSION               0.3

FROM ubuntu

# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir /.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'

EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]

# Multiple images example
#
# VERSION               0.1

FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f

FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4

# You᾿ll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```

### 指令

#### FROM

格式为 

```dockerfile
FROM <image>
FROM <image>:<tag>
```

第一条指令必须为 `FROM` 指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个 `FROM` 指令（每个镜像一次）。

#### MAINTAINER

格式为 `MAINTAINER <name>`，指定维护者信息。

#### RUN

格式有两种

```dockerfile
RUN <command>
RUN ["executable", "param1", "param2"]`
```

:warning: 前者将在 shell 终端中运行命令，即 `/bin/sh -c`；后者则使用 `exec` 执行。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]`。

**每条 `RUN` 指令将在当前镜像基础上执行指定命令**，并提交为新的镜像。当命令较长时可以使用 `\` 来换行。

#### CMD

**指定启动容器时执行的命令**，:warning: 每个 Dockerfile 只能有一条 `CMD` 命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。

**支持三种格式**

```dockerfile
CMD ["executable","param1","param2"] # 使用 exec 执行，推荐方式
CMD command param1 param2 # 在 /bin/sh 中执行，提供给需要交互的应用
CMD ["param1","param2"] # 提供给 ENTRYPOINT 的默认参数
```

#### EXPOSE

格式为 `EXPOSE <port> [<port>...]`。

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 `-P`，Docker 主机会自动分配一个端口转发到指定的端口。

#### ENV

格式为 `ENV <key> <value>`。 指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持。

例如

```dockerfile
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

#### ADD

格式为 `ADD <src> <dest>`。

该命令将复制指定的 `<src>` 到容器中的 `<dest>`。 其中 `<src>` 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）

#### COPY

格式为 `COPY <src> <dest>`。

复制本地主机的 `<src>`（为 Dockerfile 所在目录的相对路径）到容器中的 `<dest>`。

当使用本地目录为源目录时，推荐使用 `COPY`。

#### ENTRYPOINT

两种格式：

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2 #（shell中执行）
```

配置容器启动后执行的命令，并且不可被 `docker run` 提供的参数覆盖。

:warning: 每个 Dockerfile 中只能有一个 `ENTRYPOINT`，当指定多个时，只有最后一个起效。

#### VOLUME

格式为 `VOLUME ["/data"]`。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

#### USER

格式为 `USER daemon`。

指定运行容器时的用户名或 UID，后续的 `RUN` 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`。要临时获取管理员权限可以使用 `gosu`，而不推荐 `sudo`。

#### WORKDIR

格式为 `WORKDIR /path/to/workdir`。

为后续的 `RUN`、`CMD`、`ENTRYPOINT` 指令配置工作目录。

可以使用多个 `WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

则最终路径为 `/a/b/c`。

#### ONBUILD

格式为 `ONBUILD [INSTRUCTION]`。

**配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。**

例如，Dockerfile 使用如下的内容创建了镜像 `image-A`。

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

如果基于 image-A 创建新的镜像时，新的Dockerfile中使用 `FROM image-A`指定基础镜像时，会自动执行 `ONBUILD` 指令内容，等价于在后面添加了两条指令。

```
FROM image-A

#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
```

使用 `ONBUILD` 指令的镜像，推荐在标签中注明，例如 `ruby:1.9-onbuild`。

### 使用Dockerfile创建镜像

编写完成 Dockerfile 之后，可以通过 `docker build` 命令来创建镜像。

基本的格式为 `docker build [选项] 路径`，该命令将读取指定路径下（包括子目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。也可以通过 `.dockerignore` 文件（每一行添加一条匹配模式）来让 Docker 忽略路径下的目录和文件。

要指定镜像的标签信息，可以通过 `-t` 选项，例如

```shell
docker build -t myrepo/myapp /tmp/test1/
```







## WEB 应用案例

### 运行一个WEB应用

在容器内运行一个 Python Flask 应用

```shell
docker pull training/webapp
docker run -d -P training/webapp python app.py
```

+ `-d` : 让容器后台运行
+ `-P` : 将容器内部使用的网络端口映射到我们使用的主机上(随机)

```shell
docker run -d -p 8000:5000 training/webapp python app.py
```

+ `-p` : 将容器内部使用的端口映射到我们使用 的主机上(将容器内的5000端口映射到本地主机的8000端口上)

### 查看 WEB 应用容器

```shell
docker ps
```

```
CONTAINER ID        IMAGE               COMMAND             PORTS
184e850e9001        training/webapp     "python app.py"     0.0.0.0:8000->5000/tcp
dad8d5714980        training/webapp     "python app.py"     0.0.0.0:32768->5000/tcp
```

这里多了个端口信息

```
PORTS
0.0.0.0:8000->5000/tcp
0.0.0.0:32768->5000/tcp
```

Docker 开放了 5000 端口(默认Python Flask端口) 映射到主机端口32768和8000上。

这时可以通过 http://192.168.99.100:32768/ 和 http://192.168.99.100:8000/ 访问WEB应用。192.168.99.100是我的vm虚拟机的ip。

### 快捷查看网络端口

```shell
docker port <name/id>
5000/tcp -> 0.0.0.0:8000
```

### 查看 WEB 应用程序日志

```shell
docker logs -f <name/id>
```

```
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
192.168.99.1 - - [30/Aug/2019 03:52:56] "GET / HTTP/1.1" 200 -
192.168.99.1 - - [30/Aug/2019 03:52:56] "GET /favicon.ico HTTP/1.1" 404 -
192.168.99.1 - - [30/Aug/2019 03:53:05] "GET / HTTP/1.1" 200 -
```

+ `-f` : 让 docker logs 在输出后不是直接退出，而是一直接收该容器的输出

### 查看 WEB 应用程序容器内运行的进程

查看容器内部运行的进程

```shell
docker top <name/id>
```

```
UID       PID      PPID     C      STIME      TTY     TIME          CMD
root      3445     3423     0      03:49      ?       00:00:00      python app.py
```

### 查看 Docker 容器的配置与状态信息

```shell
docker inspect <name/id>
```

```
[
    {
        "Id": "184e850e9001ae4527da694f7337c6a95c49555524d4f535072ca7f8556e367e",
        "Created": "2019-08-30T03:49:51.305996931Z",
        "Path": "python",
        "Args": [
            "app.py"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 3445,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2019-08-30T03:49:51.601670982Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        ...
]
```

### 移除 WEB 应用容器

```shell
docker stop <name/id>    # 移除容器前要先停止容器
docker rm <name/id>		 # 移除容器
```

