# 编译 Android 固件

<CodeBlockTabs defaultValue="a11">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="a11">Android 11</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="a11">

运行 Android 11.0，ROC-RK3566-PC 需要满足如下条件：
- *DDR* 至少需要 **2 GB**
- *eMMC* 至少需要 **16 GB**

## Android SDK

SDK 源码与 bundle 压缩包均存放在云盘中。

## 下载 SDK [step]

- 由于 SDK 较大，可以去下载页面选择云盘下载 [Firefly-RK356X_Android11.0_git](https://community.t-firefly.com/doc/download/103)。

- 下载完成后，在解压前先校验 MD5 码：

    ```text
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.001
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.002
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.003
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.004
    $ md5sum /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.005

    b4c3d014a688d230bb25156a4c5aea26  Firefly-RK356X_Android11.0_git_20210824.7z.001
    1ddfec58d0d69aee6029982fcbe4343c  Firefly-RK356X_Android11.0_git_20210824.7z.002
    d08b16c244545ac68b496e2980d3c6a7  Firefly-RK356X_Android11.0_git_20210824.7z.003
    2b628cc10a55214b8d9a3619673c01c3  Firefly-RK356X_Android11.0_git_20210824.7z.004
    6f7e63955c96ca3c9ba6e4e49d52c90c  Firefly-RK356X_Android11.0_git_20210824.7z.005
    ```

- 然后解压：

<Callout title="注意" type="warn">

不要在共享文件夹、挂载文件夹以及非英文目录解压 SDK，避免产生不必要的错误。

</Callout>

```bash
    $ mkdir ~/proj
    $ mv /path/to/Firefly-RK356X_Android11.0_git_20210824.7z.*  ~/proj
    $ cd ~/proj/
    $ 7z x ./Firefly-RK356X_Android11.0_git_20210824.7z.001 -oRK356X_Android11.0
    $ cd ./RK356X_Android11.0
    $ git reset --hard
```

## 更新 SDK [step]

下载 SDK 后，从 GitLab 处更新代码的方法：

```bash
#1. 进入 SDK 根目录
cd ~/proj/RK356X_Android11.0

#2. 下载远程 bundle 仓库
git clone https://gitlab.com/TeeFirefly/rk356x-android11-bundle.git .bundle

#3. 若下载仓库失败，目前 bundle 仓库占用空间较大，所以同步的时候可能会出现卡住或失败的问题，
# 可以从云盘下载 bundle 并解压到 SDK 根目录，解压指令如下：

7z x rk356x-android11-bundle.7z  -r -o. && mv rk356x-android11-bundle/ .bundle/

#4. 更新 SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步 FETCH_HEAD 到 firefly 分支
git rebase FETCH_HEAD
```

随着 SDK 的更新，bundle 也会随之越来越大，可以去下载页面选择云盘下载 [bundle](https://community.t-firefly.com/doc/download/103)。
## ROC-RK3566-PC 产品编译方法 [step]

**<font color=#ff0000 size=3>注意：若是第一次编译该Android Product，请使用公版编译命令进行一次完整编译</font>**

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">整体编译</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">分步编译</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA 编译</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">
### 编译配置总览

<Callout title="模组说明" type="info">
本节编译配置对应的模组：显示屏 DM-M10R800 V2 / DM-M10R800 V3S、双目摄像头 CAM-2MS2MF、HDMI TO MIPI_CSI 驱动板（RK628D）。
</Callout>

| 编译配置 | 模组 | DTS（`-d` 参数） | lunch（`-l` 参数） |
| --- | --- | --- | --- |
| 公版编译（HDMI） | — | `rk3566-roc-pc` | `rk3566_roc_pc-userdebug` |
| 显示屏 DM-M10R800 V2 编译（MIPI_DSI） | DM-M10R800 V2 | `rk3566-roc-pc-mipi101_M101014_BE45_A1` | `rk3566_roc_pc_mipi-userdebug` |
| 显示屏 DM-M10R800 V3S 编译（MIPI_DSI） | DM-M10R800 V3S | `rk3566-roc-pc-mipi101_BSD1218_A101KL68` | `rk3566_roc_pc_mipi-userdebug` |
| 双目摄像头 CAM-2MS2MF 编译（HDMI+CAM-2MS2MF） | CAM-2MS2MF | `rk3566-roc-pc`（需先修改 dts） | `rk3566_roc_pc-userdebug` |
| HDMI TO MIPI_CSI（RK628D）编译 | RK628D | `rk3566-roc-pc`（需先修改 dts） | `rk3566_roc_pc-userdebug` |

### 公版编译 [step]
**HDMI**
```
./FFTools/make.sh -d rk3566-roc-pc -j8 -l rk3566_roc_pc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc-userdebug
```

### 显示屏编译 [step]

**DM-M10R800 V2**

**MIPI_DSI**
```
./FFTools/make.sh -d rk3566-roc-pc-mipi101_M101014_BE45_A1 -j8 -l rk3566_roc_pc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc_mipi-userdebug
```

**DM-M10R800 V3S**

**MIPI_DSI**
```
./FFTools/make.sh -d rk3566-roc-pc-mipi101_BSD1218_A101KL68 -j8 -l rk3566_roc_pc_mipi-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc_mipi-userdebug
```

### 摄像头编译 [step]
**HDMI+CAM-2MS2MF**
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts
    index 3fd84cea761..715cf0cceeb 100644
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts
    @@ -12,8 +12,8 @@
      * using single camera xc7160 ----> rk3566-roc-pc-cam-8ms1m.dtsi
      * using dual camera gc2053/gc2093   ----> rk3566-roc-pc-cam-2ms2m.dtsi
      */
    -#include "rk3566-roc-pc-cam-8ms1m.dtsi"
    -//#include "rk3566-roc-pc-cam-2ms2m.dtsi"
    +//#include "rk3566-roc-pc-cam-8ms1m.dtsi"
    +#include "rk3566-roc-pc-cam-2ms2m.dtsi"
    ```
* 编译

    ```
    ./FFTools/make.sh -d rk3566-roc-pc -j8 -l rk3566_roc_pc-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc-userdebug
    ```

### RK628D 编译 [step]
* 修改 dts

    ```
    diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts b/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts
    index 1a274d6..12805d3 100644
    --- a/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts
    +++ b/kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc.dts
    @@ -13,9 +13,9 @@
      * using dual camera gc2053/gc2093   ----> rk3566-roc-pc-cam-2ms2m.dtsi
      * using rk628d   ----> rk3566-roc-pc-tf-hdmi-mipi-rk628.dtsi
      */
    -#include "rk3566-roc-pc-cam-8ms1m.dtsi"
    +//#include "rk3566-roc-pc-cam-8ms1m.dtsi"
    //#include "rk3566-roc-pc-cam-2ms2m.dtsi"
    -//#include "rk3566-roc-pc-tf-hdmi-mipi-rk628.dtsi"
    +#include "rk3566-roc-pc-tf-hdmi-mipi-rk628.dtsi"
    ```

* 编译

    ```
    ./FFTools/make.sh -d rk3566-roc-pc -j8 -l rk3566_roc_pc-userdebug
    ./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc-userdebug
    ```

编译完成后，将在 rockdev/Image-XXX/ 目录下生成统一固件 update.img。


  </CodeBlockTab>

  <CodeBlockTab value="partial">

## 手动编译 ROC-RK3566-PC Android 11.0 [step]

编译前执行如下命令配置环境变量：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* 编译 kernel：

```
cd ~/proj/RK356X/kernel/
make ARCH=arm64 firefly_defconfig android-11.config rk356x.config
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3566_roc_pc/boot.img rk3566-roc-pc.img -j8
```

<Callout title="注意" type="info">

进行内核 debug 的时候，单编译生成 boot.img 如果出现运行错误，可参考[FAQs](../../faqs.md)。

</Callout>

* 编译 uboot：

```
cd ~/proj/RK356X/u-boot/
./make.sh rk3566
```

* 编译 Android：

```
cd ~/proj/RK356X/
source build/envsetup.sh
lunch rk3566_roc_pc-userdebug
make installclean
make -j8
./mkimage.sh
```

## 打包成统一固件 update.img

编译完可以用Firefly官方的脚本打包成统一固件，执行如下命令：
```
./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc-userdebug
```
打包完成后将在rockdev/Image-XXX/ 目录下生成统一固件： product名XXX_XXX_日期XXX.img

在 Windows 下打包统一固件 update.img 也很简单，将编译生成的文件拷贝到 AndroidTool 的 rockdev\Image 目录中，然后运行 rockdev 目录下的 mkupdate.bat 批处理文件即可创建 update.img 并存放到 rockdev\Image 目录里。

  </CodeBlockTab>

  <CodeBlockTab value="ota">
## OTA 编译

### Firefly 完整OTA编译和升级 [step]
**添加-o 参数进行OTA全局编译**

```bash
./FFTools/make.sh -d rk3566-roc-pc -j8 -l rk3566_roc_pc-userdebug -o
```

**生成完整包路径**

```text
out/target/product/rk3566_roc_pc/rk3566_roc_pc-ota-eng.xxx.zip
```

**生成基础素材包路径**

```text
out/target/product/rk3566_roc_pc/obj/PACKAGING/target_files_intermediates/rk3566_roc_pc-target_files-eng.xxx.zip
```
拷贝基础素材包并重新命名，主要用在差分包升级。

```bash
cp out/target/product/rk3566_roc_pc/obj/PACKAGING/target_files_intermediates/rk3566_roc_pc-target_files-eng.xxx.zip ~/ota/v1.zip
```

**打包编译的固件和升级**

```bash
./FFTools/mkupdate/mkupdate.sh -l rk3566_roc_pc-userdebug
```

在 `rockdev/Image-rk3566_roc_pc/` 目录下找到对应的固件烧录到机器，此后机器就可以进行全量包升级和差分包升级。将刚刚编译的完整包 `rk3566_roc_pc-ota-eng.xxx.zip` 命名为 update.zip，放到 /sdcard/ 目录下重启，并按照提示进行全量包升级。

### OTA 差分包生成 [step]

**生成新的基础素材包**

如上已经制作了 v1.zip 的基础素材包，需要再做一个 v2.zip 的素材包才能生成差分包，需修改内核 dts 相关代码或 Android 的代码，然后重新执行命令。

```bash
./FFTools/make.sh -d rk3566-roc-pc -j8 -l rk3566_roc_pc-userdebug -o
```

重新生成新的基础素材包我们可以进行拷贝。

```bash
cp out/target/product/rk3566_roc_pc/obj/PACKAGING/target_files_intermediates/rk3566_roc_pc-target_files-eng.xxx.zip  ~/ota/v2.zip
```

**生成差分包命令**

```bash
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```

**升级OTA差分包**

把 update.zip 放到机器 sdcard 目录下重启，就会自动识别到，然后按步骤点击升级即可完成。

### OTA 服务器搭建

参考[OTA 文档]()搭建服务器。
  </CodeBlockTab>
</CodeBlockTabs>

  </CodeBlockTab>
</CodeBlockTabs>
