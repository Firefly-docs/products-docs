## Build a distributed compilation environment with Docker

`distcc` is a program used for distributed compilation of C, C++, Objective-C or Objective-C++ code through several machines on the network. `distcc` does not require all machines to share the file system, have synchronized clocks, or have the same libraries and header files installed, as long as the server machine has a suitable compilation tool to compile. This example uses Docker technology to deploy the `distcc` distributed compilation service on two Firefly-RK3399 devices (arm64) and one PC (x86_64), and then realizes the use of `distcc`'s distributed compilation features on one of the Firefly-RK3399 devices To speed up the kernel compilation process.

### Ready to work

* Two Firefly-RK3399 devices;
* Router, network cable;
* PC machine.

Connect the two devices and the PC to the same LAN. The corresponding IP address after connection is:

* PC: 192.168.1.100
* Device A: 192.168.1.101
* Device B: 192.168.1.102

### PC deployment Docker

Install Docker using script:

```
wget -qO- https://get.docker.com/ | sh
```

In order to enable current ordinary users to execute docker related commands, the current user needs to be added to the dokcer user group:

```bash
sudo groupadd docker # add docker user group
sudo gpasswd -a $USER docker # Add the current user to the docker user group
newgrp docker # Update docker user group
```

Start the Docker background service:

```
sudo service docker start
```

Create a new `Dockerfile_distcc.x86_64` file. The contents of the file are as follows:

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
CMD ["--no-detach", "--log-stderr", "--log-level", "debug", "--allow", "0.0.0.0/0"]
```

Generate Docker image:

```
docker build -t distcc_server:x86_64 -f Dockerfile_distcc.x86_64.
```

You can use the command `docker images` to view the generated Docker image:

```
docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
distcc_server x86_64 138c0b7e3801 9 minutes ago 66.1MB
```

Start the Docker container with the newly created image, and provide the distcc service on the 3632 TCP port of the host network:

```
docker run -d -p 3632:3632 distcc_server:x86_64
```

Use the command `docker ps` to view the running container:

```
docker ps
CONTAINER ID    IMAGE                  COMMAND                  CREATED         STATUS          PORTS                    NAMES
fa468d068185    distcc_server:x86_64   "/usr/bin/distccd --…"   9 minutes ago   Up 9 minutes    0.0.0.0:3632->3632/tcp   epic_chatterjee
```

### Device deployment Docker

```
apt-get update
wget -qO- https://get.docker.com/ | sh
```

Add the current user to the dokcer user group:

```bash
sudo groupadd docker
sudo gpasswd -a $USER docker
newgrp docker # Update docker
```

Start the Docker background service:

```
service docker start
```

Create a new `Dockerfile_distcc.arm64` file with the following content:

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
CMD ["--no-detach", "--log-stderr", "--log-level", "debug", "--allow", "0.0.0.0/0"]
```

Generate Docker image:

```
docker build -t distcc_server:arm64 -f Dockerfile_distcc.arm64.
```

Start the Docker container with the newly created image, and provide the distcc service on the 3632 TCP port of the host network:

```
docker run -d -p 3632:3632 distcc_server:arm64
```

Use the `docker save` command to export the newly created image:

```
docker save -o distcc_server.tar distcc_server:arm64
```

Copy the generated `distcc_server.tar` file to another device, and then import the image:

```
docker load -i distcc_server.tar
```

After importing the image, there is also a `distcc_server:arm64` image on this device, and then you can run a container:

```
docker run -d -p 3632:3632 distcc_server:arm64
```

**Tip: If the user has registered a Docker Hub account, he can use `docker push` to push the image to the remote warehouse, and use `docker pull` on another device to pull the image of the remote warehouse. Users can understand related operations by themselves**.

### The client compiles the kernel with distcc

So far, all three machines have deployed a distributed compilation environment. You can choose any machine as the client, and the remaining two machines as the server. Choose one of the devices as the client.

Create a `Dockerfile_compile.arm64` file in the client to generate a Docker image for compiling the kernel. The content of the file is as follows:

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

Then generate a mirror:

```
docker build -t compile:arm64 -f Dockerfile_compile.arm64.
```

Pull the kernel file to the client before starting the container. Then create the `/etc/distcc/hosts` file, its content is to list all the IP addresses of the machines that provide distcc services. The content is as follows:

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

In order to test the results more accurately, first clear the cache on the client device:

```
echo 3> /proc/sys/vm/drop_caches
```

Start a Docker container with the `compile:arm64` image, and at the same time mount the host's current kernel directory to the `/mnt` directory in the container, and mount the host's `/etc/distcc/hosts` file to the container`/etc /distcc/hosts` and use the parameter `-it` to start the container in interactive mode:

```bash
docker run -it --rm -v $(pwd):/mnt -v /etc/distcc/hosts:/etc/distcc/hosts compile:arm64 /bin/bash
root@f4415264351b:/# ls
bin boot dev etc home lib media mnt opt proc root run sbin srv sys tmp usr var
root@f4415264351b:/# cd mnt/
```

Enter the `/mnt` directory in the container, and then start to compile the kernel with distcc, add the `time` command to view the execution time of the compilation command, the `CC` parameter indicates to use `distcc` to compile:

```
time make ARCH=arm64 rk3399-firefly.img -j32 CC="distcc"
```

**Note:** If you use a PC as the client, you need to use the following command to compile:

```
time make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rk3399-firefly.img -j32 CC="distcc aarch64-linux-gnu-gcc"
```

During the compilation process, you can use the command `distccmon-text 1` to view the compilation status in a new window in the container used for compilation:

```
distccmon-text 1
                    ...
15713 Compile perf_regs.c 192.168.1.101[0]
15327 Compile fork.c 192.168.1.101[2]
15119 Compile dma-mapping.c 192.168.1.101[3]
15552 Compile signal32.c 192.168.1.100[0]
15644 Compile open.c 192.168.1.100[2]
15112 Compile traps.c 192.168.1.100[3]
15670 Compile arm64ksyms.c 192.168.1.102[0]
15629 Compile mempool.c 192.168.1.102[2]
15606 Compile filemap.c 192.168.1.102[3]
15771 Preprocess localhost[0]
15573 Preprocess localhost[1]
15485 Preprocess localhost[2]
                    ...
```

After the final compilation command is completed, you can see the time taken for compilation:

```
real 15m44.809s
user 16m0.029s
sys 6m22.317s
```

The following is the time it takes to compile the kernel with a single device:

```
real 23m33.002s
user 113m2.615s
sys 9m29.348s
```

By comparison, it can be seen that the distributed compilation implemented by distcc can effectively improve the compilation speed.

**note:**

* Different platforms require different compilers. For example, on the x86_64 platform, you need to install the corresponding cross-compilation tool `gcc-aarch64-linux-gnu`. On the arm64 platform, you only need to install the `gcc` compilation tool. Users need to specify and install the correct tool in the corresponding `Dockerfile` file according to the actual situation.
* If you use a PC as a client, you must use the parameter `CC="distcc aarch64-linux-gnu-gcc"` to specify the use of `aarch64-linux-gnu-gcc` when compiling the kernel in the container used for compilation Compile with cross-compiler tools.