## 使用 Docker 搭建分布式编译环境

`distcc` 是个通过网络上的若干台机器用来分布式编译 C、C++、Objective-C 或 Objective-C++ 代码的程序。`distcc` 不要求所有的机器共享文件系统，有同步的时钟，或者安装有同样的库和头文件，只要作为服务器的机器有合适的编译工具即可进行编译。本例在两个 Firefly-RK3399 设备 (arm64) 和一台 PC 机 (x86_64) 上利用 Docker 技术来布署 `distcc` 分布式编译服务，进而实现在其中一个 Firefly-RK3399 板卡上利用 `distcc` 的分布式编译特性来加速内核的编译过程。

### 准备工作

*   两个 Firefly-RK3399 设备
*   路由器、网线
*   PC 机

将两块设备和 PC 机都连接到同一个局域网内。连接好后对应的 IP 地址别是：

*   PC 机：192.168.1.100
*   设备 A：192.168.1.101
*   设备 B：192.168.1.102

### PC 机部署 Docker

使用脚本安装 Docker：

```bash
wget -qO- https://get.docker.com/ | sh
```

为了可以使当前普通用户可以执行 docker 相关命令，需要将当前用户添加到 dokcer 用户组中：

```bash
sudo groupadd docker            # 添加 docker 用户组
sudo gpasswd -a $USER docker    # 将当前用户添加到 docker 用户组中
newgrp docker                   # 更新 docker 用户组
```

启动 Docker 后台服务：

```bash
sudo service docker start
```

新建一个 `Dockerfile_distcc.x86_64` 文件。其文件内容如下：

```
FROM ubuntu:bionic
MAINTAINER Firefly <service@t-firefly.com>

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests\
	gcc-aarch64-linux-gnu distcc\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists

RUN ln -s aarch64-linux-gnu-gcc /usr/bin/gcc &&\
ln -s aarch64-linux-gnu-gcc /usr/bin/cc

EXPOSE 3632

ENTRYPOINT ["/usr/bin/distccd"]
CMD ["--no-detach" , "--log-stderr" , "--log-level", "debug", "--allow" , "0.0.0.0/0"]
```

生成 Docker 镜像：

```bash
docker build -t distcc_server:x86_64 -f Dockerfile_distcc.x86_64 .
```

可以用命令 `docker images` 查看生成的 Docker 镜像：

```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
distcc_server       x86_64              138c0b7e3801        9 minutes ago       66.1MB
```

用新建的镜像启动 Docker 容器，在主机网络的 3632 TCP 端口提供 distcc 服务：

```bash
docker run -d -p 3632:3632 distcc_server:x86_64
```

用命令 `docker ps` 可以查看正在运行中的容器：

```
docker ps
CONTAINER ID    IMAGE                  COMMAND                  CREATED         STATUS          PORTS                    NAMES
fa468d068185    distcc_server:x86_64   "/usr/bin/distccd --…"   9 minutes ago   Up 9 minutes    0.0.0.0:3632->3632/tcp   epic_chatterjee
```

### 设备部署 Docker

```bash
apt-get update
wget -qO- https://get.docker.com/ | sh
```

将当前用户添加到 dokcer 用户组中：

```bash
sudo groupadd docker
sudo gpasswd -a $USER docker
newgrp docker	# 更新 docker
```

启动 Docker 后台服务：

```bash
service docker start
```

新建 `Dockerfile_distcc.arm64` 文件，内容如下：

```
FROM ubuntu:bionic
MAINTAINER Firefly <service@t-firefly.com>

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
	gcc distcc\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists

EXPOSE 3632

ENTRYPOINT ["/usr/bin/distccd"]
CMD ["--no-detach" , "--log-stderr" , "--log-level", "debug", "--allow" , "0.0.0.0/0"]
```

生成 Docker 镜像：

```
docker build -t distcc_server:arm64 -f Dockerfile_distcc.arm64 .
```

用新建的镜像启动 Docker 容器，在主机网络的 3632 TCP 端口提供 `distcc` 服务：

```
docker run -d -p 3632:3632 distcc_server:arm64
```

用 `docker save` 命令导出新建的镜像：

```
docker save -o distcc_server.tar distcc_server:arm64
```

将生成的 `distcc_server.tar` 文件复制到另一块设备上，然后导入镜像：

```
docker load -i distcc_server.tar
```

导入镜像后，在这个设备上也有了 `distcc_server:arm64` 镜像，然后可以运行一个容器：

```
docker run -d -p 3632:3632 distcc_server:arm64
```

**提示：用户如果注册了 Docker Hub 帐号，可以用 `docker push` 推送镜像到远程仓库，在另一设备上用 `docker pull` 拉取远程仓库的镜像。用户可以自行了解相关操作。**

### 客户端用 distcc 编译内核

到这里，三个机器都部署了分布式编译环境。可以选择任意一台机器作为客户端，剩下两个机器作为服务器。这里选择其中一个设备作为客户端。

在客户端中创建 `Dockerfile_compile.arm64` 文件，生成一个编译内核的 Docker 镜像，文件内容如下：

```
FROM ubuntu:bionic
MAINTAINER Firefly <service@t-firefly.com>

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends --no-install-suggests \
	bc make python sed libssl-dev binutils build-essential distcc\
    liblz4-tool gcc \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists
```

然后生成镜像：

```
docker build -t compile:arm64 -f Dockerfile_compile.arm64 .
```

在启动容器前先将内核文件拉到客户端中。然后创建 `/etc/distcc/hosts` 文件，其内容是列举出所有提供 distcc 服务的机器的 IP 地址。内容如下：

```bash
# As described in the distcc manpage, this file can be used for a global
# list of available distcc hosts.
#
# The list from this file will only be used, if neither the
# environment variable DISTCC_HOSTS, nor the file $HOME/.distcc/hosts
# contains a valid list of hosts.
#
# Add a list of hostnames in one line, seperated by spaces, here.
192.168.1.100
192.168.1.101
193.168.1.102
```

为了测试结果更准确，先清除客户端设备上的缓存：

```
echo 3 > /proc/sys/vm/drop_caches
```

用 `compile:arm64` 镜像启动一个 Docker 容器，同时将主机当前内核目录挂载到容器中的 `/mnt` 目录，将主机的 `/etc/distcc/hosts` 文件挂载到容器的 `/etc/distcc/hosts` 中，并用参数 `-it` 以交互模式启动容器：

```
docker run -it --rm -v $(pwd):/mnt -v /etc/distcc/hosts:/etc/distcc/hosts compile:arm64 /bin/bash
root@f4415264351b:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@f4415264351b:/# cd mnt/
```

进入到容器中的 `/mnt` 目录，然后开始用 distcc 编译内核，加入 `time` 命令可以查看编译命令执行耗时，`CC` 参数指明用 `distcc` 进行编译：

```
time make ARCH=arm64 rk3399-firefly.img -j32 CC="distcc"
```

**注意：如果用 PC 机作为客户端，则需要用以下命令进行编译：**

```
time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rk3399-firefly.img -j32 CC="distcc aarch64-linux-gnu-gcc"
```

编译过程中，可以在用于编译的容器内新的窗口中用命令 `distccmon-text 1` 查看编译情况：

```
distccmon-text 1
                    ...
15713  Compile     perf_regs.c                              192.168.1.101[0]
15327  Compile     fork.c                                   192.168.1.101[2]
15119  Compile     dma-mapping.c                            192.168.1.101[3]
15552  Compile     signal32.c                               192.168.1.100[0]
15644  Compile     open.c                                   192.168.1.100[2]
15112  Compile     traps.c                                  192.168.1.100[3]
15670  Compile     arm64ksyms.c                             192.168.1.102[0]
15629  Compile     mempool.c                                192.168.1.102[2]
15606  Compile     filemap.c                                192.168.1.102[3]
15771  Preprocess                                                localhost[0]
15573  Preprocess                                                localhost[1]
15485  Preprocess                                                localhost[2]
                    ...
```

最后编译命令完成后可以看到编译所用时间：

```
real    15m44.809s
user    16m0.029s
sys     6m22.317s
```

下边是单独使用一块设备进行内核编译所耗费的时间:

```
real    23m33.002s
user    113m2.615s
sys     9m29.348s
```

对比可见，用采用 distcc 实现的分布式编译可以有效提高编译速度。

**注意：**

* 平台不同，所需的编译器也不同。如在 x86_64 平台上，需要安装对应的交叉编译工具 `gcc-aarch64-linux-gnu`。在 arm64 平台则只需要安装 `gcc` 编译工具即可。用户需要根据实际情况在对应的 `Dockerfile` 文件里指定安装正确的工具。
* 如果用 PC 机做客户端，则在用于编译的容器中编译内核的时候，要用参数 `CC="distcc aarch64-linux-gnu-gcc"` 指明用 `aarch64-linux-gnu-gcc` 交叉编译工具编译。