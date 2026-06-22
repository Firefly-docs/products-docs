# Buildroot development

Buildroot is a framework for building embedded Linux systems on the Linux platform. The entire Buildroot is composed of a Makefile (*.mk) script and a Kconfig (Config.in) configuration file. You can compile a complete Linux system software (including boot, kernel, rootfs, and various libraries and applications in rootfs) that can be directly flashed to the machine through the buildroot configuration and menuconfig modification, just like compiling the Linux kernel.

[Buildroot official website](https://buildroot.org/)

[Buildroot Development Manual](https://buildroot.org/downloads/manual/manual.html)

## Directory structure

```bash
buildroot/
├── arch                # Construction and configuration files of CPU architecture
├── board               # Documents related to specific boards
├── boot                # Construction and configuration files of Bootloaders
├── build
├── CHANGES             # Buildroot modification log
├── Config.in
├── Config.in.legacy
├── configs             # Buildroot configuration file of the specific board
├── COPYING
├── DEVELOPERS
├── dl                  # Downloaded programs, source code compressed packages, patches, etc.
├── docs                # Documentation
├── fs                  # Construction and configuration files of various filesystems
├── linux               # Construction and configuration files of Linux
├── Makefile
├── Makefile.legacy
├── output              # Compile output directory
├── package             # Construction and configuration files of all packages
├── README              # Simple instructions for Buildroot
├── support             # Scripts and configuration files that provide functional support for Bulidroot
├── system              # Construction and configuration files of making root filesystem
├── toolchain           # Construction and configuration files of cross-compilation toolchain
└── utils               # Utilities
```

## Configuration

Select the default profile:

```bash
# Enter the SDK root directory
cd path/to/SDK/
# Select configuration file
# `configs/_defconfig`
source envsetup.sh 
```

After the execution is completed, a compilation output directory, `output/` will be generated, and subsequent operations of `make` can be executed in this directory.

### Configure package

Open the configuration interface:

```bash
make menuconfig
```

![](../../../rv1126b_img/linux_buildroot_menuconfig.png)

We can add or cut some tools in the configuration interface to customize system functions as required. Take adding `qt53d` as an example:

Enter `/` to enter the search interface, enter the content you want to find `qt53d`, and press Enter to search:

![](../../../rv1126b_img/linux_buildroot_serach_qt53d.png)

![](../../../rv1126b_img/linux_buildroot_qt53d.png)

Select `1` to jump to the corresponding page, press the space to select the configuration:

![](../../../rv1126b_img/linux_buildroot_select_qt53d.png)

After the configuration is completed, move to `Save` and press Enter to save to `.config`; move to `Exit` and press Enter to exit.

![](../../../rv1126b_img/linux_buildroot_save_exit.png)

Save the configuration file:

```bash
make savedefconfig
```

Save the changes to the configuration file `configs/_defconfig`.

### Configure Busybox

Open the configuration interface and configure:

```bash
make busybox-menuconfig
```

![](../../../rv1126b_img/linux_buildroot_busybox_menuconfig.png)

After the configuration is complete, move to `Exit` and press Enter to exit, select `Yes` in the pop-up window and save it to `.config`.

![](../../../rv1126b_img/linux_buildroot_busybox_save.png)

Save the configuration file:

```bash
make busybox-update-config
```

Save the changes to the configuration file `board/rockchip/common/base/busybox.config`.

## Compile

After configuring Buildroot, run `make` directly to compile.

### Compilation instructions

When you run `make` to compile, the following process will be executed:

1. Download the source code;
2. Configure, compile and install the cross-compilation toolchain;
3. Configure, compile and install the selected software package;
4. Generate the root filesystem according to the selected format;

More usages of `make` can be obtained through `make help`.

### Compile the package

We can execute `make <package>` to compile a package separately. The compilation of the software package mainly includes the process of downloading, decompressing, patching, configuring, compiling, and installing. For details, please refer to `package/pkg-generic.mk`.

* Download

  Buildroot will automatically obtain the corresponding software packages from the Internet according to the configuration `package/<package>/<package>.mk`, including some third-party libraries, plug-ins, utilities, etc., and place them in the `dl/` directory.

* Unzip

  The package will be decompressed in the `output//build/<package>-<version>` directory.

* Patch

  Patches are placed in the `package/<packgae>/` directory, and Buildroot will apply the corresponding patches after decompressing the package. If you want to modify the source code, you can modify it by patching.

* Configure
* Compile
* Install

  After the compilation is completed, the required compilation files will be copied to the `output//target/` directory.

For a certain package, we can call a certain step in the package construction through `make <package>-<target>`, as follows:

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

## Output directory

After the compilation is complete, a subdirectory will be generated in the compilation output directory `output/` as follows:

* `build/` contains all source files, including the required host tools and selected packages for Buildroot. This directory contains all package source code.
* `host/` Tools required for host-side compilation include cross-compilation tools.
* `images/` contains a compressed root filesystem image file.
* `staging/` This directory is similar to the directory structure of the filesystem. It contains all the header files and libraries generated by the compiler, as well as other development files, but they are not tailored and are too large to be used for the target filesystem.
* `target/` contains the complete root filesystem. Compared to `staging/`, it has no development files, no header files, and the binary files are `strip` processed.

## Cross-compilation tool

After Buildroot is compiled, it will generate a cross-compilation tool in the `output//host/` directory, which we can use to compile the target program.

* Cross-compilation tool directory

`output//host/bin/`

* Compile example hello.c

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
         printf("Hello World!\n");
         return 0;
}
```

* Compile

```bash
.../host/bin/arm-buildroot-linux-gnueabihf-gcc hello.c -o hello
```

* Run

Copy the executable program `hello` to the device, run `./hello`, you will see the printed message `Hello World!`.

## Rebuild

For specific instructions on rebuilding, you can check the document `buildroot/docs/manual/rebuilding-packages.txt`.

### Rebuild the package

During the development process, if the source code of a certain package is modified, Buildroot will not recompile the package. It can be operated as follows:

* Method one

```bash
make <package>-rebuild
```

* Method two

```bash
# Delete the compiled output directory of the package
rm -rf output//build/<package>-<version>
# Compile
make <package>
```

### Full Rebuild

Buildroot does not attempt to detect what parts of the system should be rebuilt when the system configuration is changed through +make menuconfig+, +make xconfig+, or one of the other configuration tools. In some cases, Buildroot should rebuild the entire system, in some cases, only a specific subset of packages. But detecting this in a completely reliable manner is very difficult, and therefore the Buildroot developers have decided to simply not attempt to do this.

#### When a full rebuild is necessary

* When the target architecture configuration is changed, a complete rebuild is needed;
* When the toolchain configuration is changed, a complete rebuild generally is needed;
* When an additional package is added to the configuration, a full rebuild is not necessarily needed;
* When a package is removed from the configuration, buildroot does not do anything special. it does not remove the files installed by this package from the target root filesystem or from the toolchain _sysroot_. a full rebuild is needed to get rid of this package;
* When the sub-options of a package are changed, the package is not automatically rebuilt;
* When a change to the root filesystem skeleton is made, a full rebuild is needed;

Generally speaking, when you're facing a build error and you're unsure of the potential consequences of the configuration changes you've made, do a full rebuild. Specific instructions can be found in the document `rebuilding-packages.txt`.

#### Full rebuild

* Method one

Directly delete the compilation output directory, and then re-configure and compile.

```bash
rm -rf output/
```

* Method two

Executing the following command will delete the compilation output and recompile.

```bash
make clean all
```

## Add local source package

During the development process, the built-in software package of Buildroot may not meet our needs sometimes, so we need to add a custom software package. Buildroot supports packages in a variety of formats, including generic-package, cmake-package, autotools-package, etc. We take generic-package as an example.

* Create a project directory

```bash
cd path/to/SDK/
mkdir buildroot/package/rockchip/firefly_demo/
```

* Create Config.in

Add Config.in under `firefly_demo/`:

```Kconfig
config BR2_PACKAGE_FIREFLY_DEMO
        bool "Simple Firefly Demo"
```

* Create firefly_demo.mk

Add firefly_demo.mk under `firefly_demo/`:

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

* Create source code directory

The source code directory `external/firefly_demo/src` has been specified in the above Makefile.

```bash
cd path/to/SDK/
mkdir external/firefly_demo/src
```

* Write source code firefly_demo.c

Add firefly_demo.c under `firefly_demo/src/`:

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
        printf("Hello World!\n");
        return 0;
}
```

* Write Makefile

Add Makefile under `firefly_demo/src/`:

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

* Modify the upper Config.in

Add a line at the end of `buildroot/package/rockchip/Config.in`:

```Kconfig
source "package/rockchip/firefly_demo/Config.in"
```

* Select package

Open the configuration menu `make menuconfig`, find firefly_demo and select the configuration.

* Compile

```bash
# Compile firefly_demo
make firefly_demo
# Package into the root filesystem
make
# If you modify the source code, recompile the package
make firefly_demo-rebuild
```

## rootfs-overlay

rootfs-overly is a pretty good feature, it can overwrite the specified file to a directory after the target filesystem is compiled. In this way, we can easily add or modify some files to the root filesystem.

Suppose we want to add the file `overlay-test` under the `/etc/` directory of the root filesystem, we can do as follows:

* Set rootfs-overlay root directory

Open the configuration menu `make menuconfig`, and add the root directory for coverage by setting the `BR2_ROOTFS_OVERLAY` option. For rk356x, the directory `board/rockchip/rk356x/fs-overlay/` has been added by default.

* Add files to the coverage directory

```bash
cd buildroot/board/rockchip/rk356x/fs-overlay/
mkdir etc/
touch etc/overlay-test
```

* Compile

```bash
make
```

* Download the root filesystem

Download the compiled root filesystem `output//images/rootfs.ext2` to the device. Start the device, you can see that the file `/etc/overlay-test` has been added.

You can also check the `target/` directory to verify whether the addition is successful:

```bash
ls buildroot/output//target/etc/overlay-test
```
