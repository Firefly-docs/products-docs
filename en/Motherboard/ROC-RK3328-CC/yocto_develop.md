## Yocto Development

The Yocto versions currently released and supported by firefly are:
* Yocto 2.7.4 : warrior


### I. Prepare the environment:
100G free disk space

Git version must be 1.8.3.1 or higher, tar version must be 1.28 or higher, Python3 version must be 3.5.0 or higher, gcc version must be 5.0 or higher

Execute the following command to install the dependent package:
```
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools gcc-arm-linux-gnueabihf libssl-dev liblz4-tool genext2fs lib32stdc++6 \
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo fakeroot \
libparse-yapp-perl default-jre patchutils
```

### II. Get SDK

**If you have downloaded Linux_SDK, you don't need to download Yocto, just enter the linux_sdk/yocot directory.**
```
$ repo init --no-clone-bundle --repo-url   https://gitlab.com/firefly-linux/git-repo.git  -u https://gitlab.com/firefly-linux/manifests.git -b master -m yocto_release.xml
$ repo sync -c
```


yocto SDK目录树如下：
```
.
├── bitbake -> poky/bitbake       // The production tools used by OpenEmbedded to build the system
├── build                     // User configuration file and project build output directory
├── conf
│    └── local.conf -> rk3399/firefly-rk3399.conf  //Select board profile
├── meta-browser              // Web browser recipe
├── meta-clang                    // The LLVM framework system used to build the compiler
├── meta-openembedded         // Metadata for cross-compiling, installation and packaging
├── meta-poky -> poky/meta-poky       // Configuration data of Poky release version
├── meta-python2              // Python2 recipe
├── meta-qt5                  // Qt5 formula officially launched by Qt5
├── meta-rockchip             // Rockchip layer, including Rockchip chip BSP related formula
├── meta-rust                 // OpenEmbedded/Yocto layer of Rust and Cargo
├── oe-init-build-env -> poky/oe-init-build-env       // Environment used to build OE
├── poky                      // The build system used to build Linux
└── scripts -> poky/scripts       // Script file for Poky release version
```


### III. Compile Yocto
Enter the directory <path/to/yocto> and execute the following commands in order:

```
source oe-init-build-env

# Select the configuration file of the board. (e.g firefly-rk399.conf)
# cd <path/to/yocto>/build/conf
# ln -sf firefly-rk399.conf local.conf
cd <path/to/yocto>/build
bitbake <target>
```

<target> The public options have the following options

  * core-image-minimal : A small mirror that can start the device, it has no interface
  * core-image-sato : An image that supports Sato, it supports X11 with Sato theme and Pimlico application, and also includes a terminal, editor and file manager. It is a user interface environment based on GNOME Mobile, using matchbox as a window manager
  * meta-toolchain：An option to compile the gcc cross tool chain installation program, the generated file is located in the directory <path/to/yocto>/build/tmp/deploy/sdk, and the main output file is the .sh installation file
  * meta-ide-support：An option to ensure that the directory <path/to/yocto>/build contains the IDE toolchain package, the generated file is environment-setup-xxxxxx-neon-poky-linux-gnueabi, located in the directory <path/to/yocto> /build/tmp, run it directly with the soucre command

<target> Reference documents for other options：[https://www.yoctoproject.org/docs/1.1/poky-ref-manual/poky-ref-manual.html#ref-images](https://www.yoctoproject.org/docs/1.1/poky-ref-manual/poky-ref-manual.html#ref-images)


### IV. Download firmware
Edit this area
I. Yocto firmware download:
The compiled firmware is located in the directory <path/to/yocto>/build/tmp/deploy/<MACHINE>. The files to be downloaded are .wic and update.img. After entering the loader mode, execute the following commands:

```
$ sudo upgrade_tool wl 0 <IMAGE NAME>.wic
$ sudo upgrade_tool uf update.img
```