# Kernel 使用

## 前言

Firefly kernel 没有开启全部的内核功能选项，用户在使用官方默认固件发现内核不满足使用场景时，比如 USB CAN 相关功能，则需要自行开启并重新编译、烧录内核。下面介绍方法：

## 定制

首先需要获取 SDK ，开发环境的准备和获取方法查看：

[Linux 开发](prepare_compile_linux.md)

[Android 开发](prepare_compile_android.md)

之后开始新增内核选项：

* 首先进入 SDK/kernel 文件夹，将配置文件写入 .config :
```bash
# Linux
make ARCH=arm64 firefly_linux_defconfig

# Android
make ARCH=arm64 firefly_defconfig
```

* 进入配置菜单
```bash
make ARCH=arm64 menuconfig
```
之后会进入一个图形化界面进行配置
![](../../../rk356x_img/kernel_menuconfig.jpg)

* 使用介绍

1. 选项框内为星号`*`表示开启并编译进内核，空白表示不开启，`M`表示开启并编译为模块
1. 选项框分为`[]`和`<>`,`[]`只能选择编译(按Y)或者去除(按N)，`<>`除了选择编译或者去除外还可以选择编译为模块(按M)；
1. 选项后面有`—>`，说明在改目录下还有子目录；
1. 双击`ESC`表示退出，按下`？`按键可以显示帮助信息，按下`/`按键可以输入搜索内容来全局搜索信息；
1. 上下左右四个方向键是移动光标，`Enter`是选中；

* 注意事项

选择编译为模块`(M)`需要后续进行安装才能正常使用，所以添加少量配置建议使用编译进内核方式`(Y)`，省去安装步骤

搜索结果中按下最左侧显示的数字，可以直接前往这个配置项所在位置：

![](../../../rk356x_img/kernel_menuconfig_search.jpg)

属于同一个组的多个选项建议全部打开，例如下图中所有选项都属于 CAN USB 接口支持，可以全部打开以获得完整支持

![](../../../rk356x_img/kernel_menuconfig_demo.jpg)

对选项有疑问可以移动光标到选项上，按`？`查看说明

* 保存

打开需要的配置后，按方向键左右将光标移动到`Save`并按下 3 次回车进行保存

之后持续双击`Esc`退出

退出后将修改内容保存到配置文件：
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig (Android 为 firefly_defconfig)
```

## 编译

回到 SDK 根目录进行编译：
```bash
# 选择板型配置文件
./build.sh xxxx.mk
# 编译内核
./build.sh kernel
```
生成的文件位置：`SDK/kernel/boot.img`

## 烧录

参考[分区烧写](03-upgrade_firmware.html#shao-xie-fen-qu-ying-xiang)