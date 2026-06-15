# Buildroot 开发

Buildroot 是 Linux 平台上一个构建嵌入式 Linux 系统的框架。整个 Buildroot 是由 Makefile(*.mk) 脚本和 Kconfig(Config.in) 配置文件构成的。你可以和编译 Linux 内核一样，通过 buildroot 配置，menuconfig 修改，编译出一个完整的可以直接烧写到机器上运行的 Linux 系统软件（包含 boot、kernel、rootfs 以及 rootfs 中的各种库和应用程序）。

[Buildroot 官网](https://buildroot.org/)

[Buildroot 开发手册](https://buildroot.org/downloads/manual/manual.html)

## 目录结构

```bash
buildroot/
├── arch                # CPU 架构的构建、配置文件
├── board               # 具体单板相关的文件
├── boot                # Bootloaders 的构建、配置文件
├── build
├── CHANGES             # Buildroot 修改日志
├── Config.in
├── Config.in.legacy
├── configs             # 具体单板的 Buildroot 配置文件
├── COPYING
├── DEVELOPERS
├── dl                  # 下载的程序、源码压缩包、补丁等
├── docs                # 文档
├── fs                  # 各种文件系统的构建、配置文件
├── linux               # Linux 的构建、配置文件
├── Makefile
├── Makefile.legacy
├── output              # 编译输出目录
├── package             # 所有软件包的构建、配置文件
├── README              # Buildroot 简单说明
├── support             # 为 Bulidroot 提供功能支持的脚本、配置文件
├── system              # 制作根文件系统的构建、配置文件
├── toolchain           # 交叉编译工具链的构建、配置文件
└── utils               # 实用工具
```

## 配置

选择默认配置文件：

```bash
# 进入 SDK 根目录
cd path/to/SDK/
# 选择配置文件
# `configs/_defconfig`
source envsetup.sh 
```

执行完成后会生成编译输出目录，`output/`，后续也可以在该目录下执行 `make` 相关操作。

### 软件包配置

打开配置界面：

```bash
make menuconfig
```

![](../../../rk3576_img/linux_buildroot_menuconfig.png)

我们可以在配置界面添加或裁剪一些工具，按需求定制系统功能。以添加 `qt53d` 为例：

输入 `/` 进入搜索界面，输入要查找的内容 `qt53d`，按回车进行搜索：

![](../../../rk3576_img/linux_buildroot_serach_qt53d.png)

![](../../../rk3576_img/linux_buildroot_qt53d.png)

选择 `1` 跳转到对应页面，按空格选中配置：

![](../../../rk3576_img/linux_buildroot_select_qt53d.png)

配置完成后，移动到 `Save` 按回车保存到 `.config`；移动到 `Exit` 按回车退出。

![](../../../rk3576_img/linux_buildroot_save_exit.png)

保存配置文件：

```bash
make savedefconfig
```

将修改保存到配置文件 `configs/_defconfig`。

### Busybox 配置

打开配置界面，进行配置：

```bash
make busybox-menuconfig
```

![](../../../rk3576_img/linux_buildroot_busybox_menuconfig.png)

配置完成后，移动到 `Exit` 按回车退出，在弹窗页面选择 `Yes` 保存到 `.config`。

![](../../../rk3576_img/linux_buildroot_busybox_save.png)

保存配置文件：

```bash
make busybox-update-config
```

将修改保存到配置文件 `board/rockchip/common/base/busybox.config`。

## 编译

配置好 Buildroot 后，直接运行 `make` 进行编译。

### 编译说明

运行 `make` 进行编译时，会执行以下过程：

1. 下载源码；
2. 配置、编译、安装交叉编译工具链；
3. 配置、编译、安装选择的软件包；
4. 按选择的格式生成根文件系统；

关于 `make` 的更多用法，可通过 `make help` 获得。

### 编译软件包

我们可以执行 `make <package>` 单独编译某个软件包。软件包的编译主要包括下载，解压，打补丁，配置，编译，安装等过程，具体可以查看 `package/pkg-generic.mk`。

* 下载

  Buildroot 会根据配置 `package/<package>/<package>.mk`，自动从网络获取对应的软件包，包括一些第三方库，插件，实用工具等，放在 `dl/` 目录。

* 解压

  软件包会解压在 `output//build/<package>-<version>` 目录下。

* 打补丁

  补丁集中放在 `package/<packgae>/` 目录，Buildroot 会在解压软件包后为其打上相应的补丁。如果要修改源码，可以通过打补丁的方式进行修改。

* 配置
* 编译
* 安装

  编译完成后，会将需要的编译生成文件拷贝到 `output//target/` 目录。

对于某个软件包，我们可以通过 `make <package>-<target>` 调用软件包构建中的某一步骤，如下：

```bash
Package-specific:
  <pkg>                  - Build and install <pkg> and all its dependencies
  <pkg>-source           - Only download the source files for <pkg>
  <pkg>-extract          - Extract <pkg> sources
  <pkg>-patch            - Apply patches to <pkg>
  <pkg>-depends          - Build <pkg>'s dependencies
  <pkg>-configure        - Build <pkg> up to the configure step
  <pkg>-build            - Build <pkg> up to the build step
  <pkg>-graph-depends    - Generate a graph of <pkg>'s dependencies
  <pkg>-dirclean         - Remove <pkg> build directory
  <pkg>-reconfigure      - Restart the build from the configure step
  <pkg>-rebuild          - Restart the build from the build step
```

## 编译输出目录

编译完成后，在编译输出目录 `output/` 会生成子目录，说明如下：

* `build/` 包含所有的源文件，包括 Buildroot 所需主机工具和选择的软件包，这个目录包含所有软件包源码。
* `host/` 主机端编译需要的工具，包括交叉编译工具。
* `images/` 包含压缩好的根文件系统镜像文件。
* `staging/` 这个目录类似根文件系统的目录结构，包含编译生成的所有头文件和库，以及其他开发文件，不过他们没有裁剪，比较庞大，不适用于目标文件系统。
* `target/` 包含完整的根文件系统，对比 `staging/`，它没有开发文件，不包含头文件，二进制文件也经过 `strip` 处理。

## 交叉编译工具

Buildroot 编译完成后，会在 `output//host/` 目录下，生成交叉编译工具，我们可以用来编译目标程序。

* 交叉编译工具目录

`output//host/bin/`

* 编译示例 hello.c

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
        printf("Hello World!\n");
        return 0;
}
```

* 编译

```bash
.../host/bin/arm-buildroot-linux-gnueabihf-gcc hello.c -o hello
```

* 运行

将可执行程序 `hello` 拷贝到设备，运行 `./hello`，则会看到打印信息 `Hello World!`。

## 重建

对于重建的具体说明，可以查看文档 `buildroot/docs/manual/rebuilding-packages.txt`。

### 重建软件包

在开发过程中，若修改了某个软件包的源码，Buildroot 是不会重新编译该软件包的。可以按如下方式操作：

* 方式一

```bash
make <package>-rebuild
```

* 方式二

```bash
# 删除软件包的编译输出目录
rm -rf output//build/<package>-<version>
# 编译
make <package>
```

### 完全重建

当通过 make menuconfig，make xconfig 或其他配置工具之一更改系统配置时，Buildroot 不会尝试检测应重建系统的哪些部分。在某些情况下，Buildroot 应该重建整个系统，在某些情况下，仅应重建软件包的特定子集。但是以完全可靠的方式检测到这一点非常困难，因此 Buildroot 开发人员已决定不尝试这样做。

#### 何时需要完全重建

* 更改目标体系结构配置时，需要完全重建；
* 更改工具链配置时，需要完全重建；
* 将其他软件包添加到配置中时，不一定需要完全重建；
* 从配置中删除软件包时，Buildroot 不会执行任何特殊操作。它不会从目标根文件系统或工具链中删除此软件包安装的文件。需要完全重建才能删除这些文件；
* 更改软件包的子选项时，不会自动重建软件包；
* 对根文件系统框架进行更改时，需要完全重建；

一般而言，当你遇到构建错误并且不确定所做的配置更改可能带来的后果时，请进行完全重建。具体说明可以查看文档 `rebuilding-packages.txt`。

#### 如何完全重建

* 方式一

直接删除编译输出目录，之后重新进行配置、编译。

```bash
rm -rf output/
```

* 方式二

执行如下命令，会删除编译输出并重新编译。

```bash
make clean all
```

## 新增本地源码包

开发过程中，Buildroot 自带的软件包有时可能无法满足我们的需求，为此我们需要添加自定义的软件包。Buildroot 支持多种格式的软件包，包括 generic-package、cmake-package、autotools-package 等，我们以 generic-package 举例说明。

* 创建工程目录

```bash
cd path/to/SDK/
mkdir buildroot/package/rockchip/firefly_demo/
```

* 新建 Config.in

在 `firefly_demo/` 下添加 Config.in：

```Kconfig
config BR2_PACKAGE_FIREFLY_DEMO
        bool "Simple Firefly Demo"
```

* 新建 firefly_demo.mk

在 `firefly_demo/` 下添加 firefly_demo.mk：

```makefile
##################################################
###########
#
### firefly_demo
#
##################################################
###########
ifeq ($(BR2_PACKAGE_FIREFLY_DEMO), y)

        FIREFLY_DEMO_VERSION:=1.0.0
        FIREFLY_DEMO_SITE=$(TOPDIR)/../external/firefly_demo/src
        FIREFLY_DEMO_SITE_METHOD=local

define FIREFLY_DEMO_BUILD_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) CC=$(TARGET_CC) CXX=$(TARGET_CXX) -C $(@D)
endef

define FIREFLY_DEMO_CLEAN_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) -C $(@D) clean
endef

define FIREFLY_DEMO_INSTALL_TARGET_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) -C $(@D) install
endef

define FIREFLY_DEMO_UNINSTALL_TARGET_CMDS
        $(TARGET_MAKE_ENV) $(MAKE) -C $(@D) uninstall
endef

$(eval $(generic-package))
endif
```

* 创建源码目录

上文的 Makefile 文件里已经指定了源码目录 `external/firefly_demo/src`。

```bash
cd path/to/SDK/
mkdir external/firefly_demo/src
```

* 编写源码 firefly_demo.c

在 `firefly_demo/src/` 下添加 firefly_demo.c：

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
        printf("Hello World!\n");
        return 0;
}
```

* 编写 Makefile

在 `firefly_demo/src/` 下添加 Makefile：

```makefile
DEPS =
OBJ = firefly_demo.o
CFLAGS =
%.o: %.c $(DEPS)
        $(CC) -c -o $@ $< $(CFLAGS)

firefly_demo: $(OBJ)
        $(CXX) -o $@ $^ $(CFLAGS)

.PHONY: clean
clean:
        rm -f *.o *~ firefly_demo

.PHONY: install
install:
        cp -f firefly_demo $(TARGET_DIR)/usr/bin/

.PHONY: uninstall
uninstall:
        rm -f $(TARGET_DIR)/usr/bin/firefly_demo
```

* 修改上一级 Config.in

在 `buildroot/package/rockchip/Config.in` 末尾添加一行：

```Kconfig
source "package/rockchip/firefly_demo/Config.in"
```

* 配置软件包

打开配置菜单 `make menuconfig`，找到 firefly_demo 并选中配置。

* 编译

```bash
# 编译 firefly_demo
make firefly_demo
# 打包进根文件系统
make
# 若修改源码，重新编译软件包
make firefly_demo-rebuild
```

## rootfs-overlay

rootfs-overly 是一个相当不错的功能，它能够在目标文件系统编译完成后将指定文件覆盖到某个目录。通过这种方式，我们可以方便地添加或修改一些文件到根文件系统。

假设我们要在根文件系统的 `/etc/` 目录下添加文件 `overlay-test`，可以按如下步骤操作：

* 设置 rootfs-overlay 根目录

打开配置菜单 `make menuconfig`，通过设置 `BR2_ROOTFS_OVERLAY` 选项，添加用于覆盖的根目录。对于 RK3576，默认已添加了目录 `board/rockchip/rk356x/fs-overlay/`。

* 添加文件到覆盖目录

```bash
cd buildroot/board/rockchip/rk356x/fs-overlay/
mkdir etc/
touch etc/overlay-test
```

* 编译

```bash
make
```

* 下载根文件系统

将编译好的根文件系统 `output//images/rootfs.ext2` 下载到设备。启动设备，可以看到已添加文件 `/etc/overlay-test`。

也可以通过查看 `target/` 目录，验证是否添加成功：

```bash
ls buildroot/output//target/etc/overlay-test
```
