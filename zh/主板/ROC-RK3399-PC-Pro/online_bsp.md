## 在线更新内核和 U-Boot

本小节介绍了在线更新的一个简单的流程。将内核、U-Boot 或者其他需要更新的文件打包成 deb 安装包，然后导入到本地包仓库，实现在设备上下载并自动更新。仅供用户参考。

### 准备 deb 安装包

操作中需要升级内核和 U-Boot，事先已经准备好了修改好的相关文件：`uboot.img` 、`trust.img` 、`boot.img` 。

deb 是 Debian Linux 的软件包格式，打包最关键的是在 `DEBIAN` 目录下创建一个 `control` 文件。首先创建 deb 工作目录，然后在 deb 目录中创建相应的目录和文件：

```bash
mkdir deb
cd deb
mkdir firefly-firmware    # 创建 firefly-firmware 目录
cd firefly-firmware
mkdir DEBIAN    # 创建 DEBIAN 目录，这个目录是必须要有的。
mkdir -p usr/share/{kernel,uboot}
mv ~/boot.img ~/deb/firefly-firmware/usr/share/kernel  # 将相应文件放到相应的目录
mv ~/uboot.img ~/deb/firefly-firmware/usr/share/uboot
mv ~/trust.img ~/deb/firefly-firmware/usr/share/uboot
```

`DEBIAN` 目录下存放的文件是 deb 包安装的控制文件以及相应的脚本文件。

在 `DEBIAN` 目录下创建控制文件 `control` 和脚本文件 `postinst`。

`control` 文件内容如下,用于记录软件标识，版本号，平台，依赖信息等数据。

```
Package: firefly-firmware # 文件目录名
Version: 1.0    # 版本号
Architecture: arm64 # 架构
Maintainer: neg   # 维护人员，自定义即可
Installed-Size: 1
Section: test
Priority: optional
Descriptionon: This is a deb test
```

`postinst` 文件内容如下,就是将需要更新的内核和 U-Boot 文件用 `dd` 命令写进对应分区的脚本：

```bash
echo "-----------uboot updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/uboot.img of=/dev/disk/by-partlabel/uboot

echo "-----------trust updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/trust.img of=/dev/disk/by-partlabel/trust

echo "-----------kernel updating------------"
dd conv=fsync,notrunc if=/usr/share/kernel/boot.img of=/dev/disk/by-partlabel/boot
```

说明：`postinst` 脚本，是在解包数据后运行的脚本，对应的还有:

* preinst：在解包数据前运行的脚本;
* prerm：卸载时，在删除文件之前运行的脚本;
* postrm：在删除文件之后运行的脚本;

其他相关知识点用户可以自行了解。这里只用到了 `preinst` 脚本。

以下是创建好的目录树：

```
deb
└── firefly-firmware
    ├── DEBIAN
    │   ├── control
    │   └── postinst
    └── usr
        └── share
            ├── kernel
            │   └── boot.img
            └── uboot
                ├── trust.img
                └── uboot.img
```

进入 deb 目录，用 `dpkg` 命令生成 deb 包：

```bash
dpkg -b firefly-firmware firefly-firmware_1.0_arm64.deb
```

生成 deb 包时，一般按照这种规范进行命名：`package_version-reversion_arch.deb` 。

### 创建本地仓库

首先安装需要的包：

```bash
sudo apt-get install reprepro gnupg
```

然后用 GnuPG 工具生成一个 GPG 密匙，执行命令后请根据提示操作：

```bash
gpg --gen-key
```

执行 `sudo gpg --list-keys` 可以查看刚刚生成的密匙信息:

```bash
sudo gpg --list-keys

gpg: WARNING: unsafe ownership on homedir '/home/firefly/.gnupg'
gpg: 正在检查信任度数据库
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: 深度：0 有效性：  4 已签名：  0 信任度：0-，0q，0n，0m，0f，4u
gpg: 下次信任度数据库检查将于 2021-05-29 进行
/home/firefly/.gnupg/pubring.kbx
--------------------------------
pub   rsa3072 2019-05-31 [SC] [有效至：2021-05-30]
      BCB65788541D632C057E696B8CBC526C05417B76
uid           [ 绝对 ] firefly <firefly@t-chip.com>
sub   rsa3072 2019-05-31 [E] [有效至：2021-05-30]
```

接下来创建包仓库，首先创建目录:

```bash
cd /var/www
mkdir apt   # 包仓库目录
mkdir -p ./apt/incoming
mkdir -p ./apt/conf
mkdir -p ./apt/key
```

把前面生成的密匙导出到仓库文件夹，请用户对应好自己创建的用户名和邮箱地址。

```bash
gpg --armor --export firefly firefly@t-chip.com > /var/www/apt/key/deb.gpg.key
```

在 `conf` 目录下创建 `distributions` 文件，其内容如下：

```
Origin: Neg   # 你的名字
Label: Mian     # 库的名字
Suite: stable   # (stable 或 unstable)
Codename: bionic    # 发布代码名，可自定义
Version: 1.0    # 发布版本
Architectures: arm64    # 架构
Components: main    # 组件名，如main，universe等
Description: Deb source test        # 相关描述
SignWith: BCB65788541D632C057E696B8CBC526C05417B76  # 上面步骤中生成的 GPG 密匙
```

建立仓库树：

```bash
reprepro --ask-passphrase -Vb /var/www/apt export
```

将第 2 步做好的 deb 包加入到仓库中：

```bash
reprepro --ask-passphrase -Vb /var/www/apt includedeb bionic ~/deb/firefly-firmware_1.0_arm64.deb
```

可以查看库中添加的文件：

```bash
root@Desktop:~# reprepro -b /var/www/apt/ list bionic
bionic|main|arm64: firefly-firmware 1.0
```

你的包已经加入了仓库，如果要移除它的话采用如下命令:

```bash
reprepro --ask-passphrase -Vb /var/www/apt remove bionic firefly-firmware
```

安装 nginx 服务器：

```
sudo apt-get install nginx
```

修改nginx的配置文件 `/etc/nginx/sites-available/default` 为：

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/apt;      //本地包仓库的路径

    access_log /var/log/nginx/repo.access.log;
    error_log   /var/log/nginx/repo.error.log;

    location ~ /(db|conf) {
        deny all;
        return 404;
    }
}
```

重启 nginx 服务器：

```bash
sudo service nginx restart
```

### 客户端更新安装

在客户端设备中，首先要添加本地包仓库的源，在目录 `/etc/apt/sources.list.d` 下添加一个新的配置文件 `bionic.list`，内容如下：

```bash
deb http://192.168.31.106 bionic main
```

IP 地址是服务器地址, `bionic` 是仓库发布代码名， `main` 是组件名。

从服务器中获取并添加 GPG 密匙：

```bash
wget -O - http://192.168.31.106/key/deb.gpg.key | apt-key add -
```

更新后即可安装自定义软件源里的 `firefly-firmware_1.0_arm64` 包：

```bash
root@firefly:/home/firefly# apt-get update
Hit:1 http://192.168.31.106 bionic InRelease
Hit:2 http://wiki.t-firefly.com/firefly-rk3399-repo bionic InRelease
Hit:3 http://ports.ubuntu.com/ubuntu-ports bionic InRelease
Hit:4 http://archive.canonical.com/ubuntu bionic InRelease
Hit:5 http://ports.ubuntu.com/ubuntu-ports bionic-updates InRelease
Hit:6 http://ports.ubuntu.com/ubuntu-ports bionic-backports InRelease
Hit:7 http://ports.ubuntu.com/ubuntu-ports bionic-security InRelease
Reading package lists... Done

root@firefly:/home/firefly# apt-get install firefly-firmware
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libllvm6.0
Use 'apt autoremove' to remove it.
The following NEW packages will be installed:
  firefly-firmware
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 0 B/6982 kB of archives.
After this operation, 1024 B of additional disk space will be used.
Selecting previously unselected package firefly-firmware.
(Reading database ... 117088 files and directories currently installed.)
Preparing to unpack .../firefly-firmware_1.0_arm64.deb ...
Unpacking firefly-firmware (1.0) ...
Setting up firefly-firmware (1.0) ...
-----------uboot updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.437281 s, 9.6 MB/s
-----------trust updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.565762 s, 7.4 MB/s
-----------kernel updating------------
39752+0 records in
39752+0 records out
20353024 bytes (20 MB, 19 MiB) copied, 0.1702 s, 120 MB/s
```

可以看到安装过程中执行了 deb 包中的 `poinst` 脚本。安装完成后，重启设备，更新完成。

在 `/usr/share` 目录中还可以看到 `kernel` 和 `uboot` 目录，分别存放了 `boot.img`，`uboot.img` 和 `trust.img` 。

注意事项

* 在制作 deb 包中，与 `DEBIAN` 同级的目录视为根目录，即放在与 `DEBIAN` 同目录下的文件，在客户端安装 deb 包后，可以在客户端的根目录下找到
* deb 包中的文件和脚本，用户要根据自己的实际情况做调整
* 每次在仓库中修改了配置文件后，都要重新导入仓库目录树
* nginx 服务器配置中，`root` 参数配置的是仓库的地址，请用户根据自己的实际情况修改
* 客户端添加的新下载源的文件时，注意核对正确的服务器地址，包仓库代码名和组件名。注意客户端要连通服务器
* 客户端要用 `apt-key add` 命令添加 GPG 密匙之后，才可以获取本地的仓库的信息