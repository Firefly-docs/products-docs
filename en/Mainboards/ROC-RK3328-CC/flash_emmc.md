# Flashing to the eMMC

## Boot Mode

eMMC flash is commonly soldered directly to the board. Some eMMC flash are pluggable, but it is hard to find a reader to use on PC. Therefore, eMMC is generally flashed onboard, that is, running to tiny system on the obard, which reads firmware data from PC and flashes to eMMC.

Depending on the existing data on the eMMC flash, there are two special boot modes: [Rockusb Mode] and [Maskrom Mode].

You usually just need to enter [Rockusb Mode] for upgrading an existing Android OS or Firefly Ubuntu OS, which is packed with [RK Firmware] format.

[Maskrom Mode] is the last resort when [Rockusb Mode] is not available due to bootloader damage, or you need to flash [Raw Firmware] to eMMC.

### Rockusb Mode

If the board powers on and finds a valid IDB (IDentity Block) in the eMMC, it will continue to load the bootloader from the eMMC and pass execution control to it. If the bootloader checks that the Recovery button is pressed and USB connection is made, then it enters the so-called [Rockusb Mode], waiting for further instructions from the host.

Requirement:

- 5V2A power adapter.
- Micro USB cable to connect power adapter and board.
- Male to male USB cable to connect host PC and board.
- eMMC.

Steps:

1. Pull all the USB cables (including micro USB cable and male to male USB cable) out of the board, to keep the board powering off.
2. Install the eMMC and pull out the SD card.
3. Use the male to male USB cable to connect the host PC with the USB 2.0 OTG port (the lower one in the double-decker ports) of the board.

    ![](../../../rk3328_img/ROC-RK3328-CC/hw_board_usbconn.png)

4. Keep the RECOVERY button on the board pressed.
5. Plug in the micro USB cable to the board to power up.
6. Wait about 3 seconds before releasing the RECOVERY button.

### Maskrom Mode

If anyone of the following conditions is met when the board powers on:

- eMMC is empty.
- The bootloader on eMMC is damaged.
- eMMC read data failed by connecting eMMC data/clock pin to ground.

then no valid IDB (IDentity Block) will be found in the eMMC. The CPU will execute a small ROM code, waiting for the host to upload via USB a small DDR blob to initialize the DDR memory, and later a full bootloader to handle further firmware upgrading. This mode is called [Maskrom Mode].

It involves hardware operation to force into [MaskRom Mode], which has certain risk and should be carried out **VERY CAREFULLY**.

Requirement:

- 5V2A power adapter.
- Micro USB cable to connect power adapter and board.
- Male to male USB cable to connect host PC and board.
- Metal tweezers to connect eMMC clock pin to ground.
- eMMC.

Steps:

1. Pull all the USB cables (including micro USB cable and male to male USB cable) out of the board, to keep the board power off.
2. Install the eMMC and pull out the SD card.
3. Use a male to male USB cable to connect your host PC and USB OTG port of the board:

    ![](../../../rk3328_img/ROC-RK3328-CC/hw_board_usbconn.png)

4. Find the reserved eMMC CLK and GND pads on the board, as shown below:

    ![](../../../rk3328_img/ROC-RK3328-CC/rk3328_maskrom_pads.jpg)

5. Connect the eMMC CLK and GND pads with metal tweezers and keep holding steadily.
6. Plug in the micro USB cable to the board to power on.
7. Wait about 1 seconds before releasing the metal tweezers.

## Download Firmware

[Firmware Download Page](http://en.t-firefly.com/doc/download/page/id/34.html)

***Firmware description***: the firmware is classified into [Raw Firmware] and [RK Firmware], which are placed into different folders. The latest firmware is the one with the latest date, which is often more stable. Please choose the correct tool to flash according to the type of firmware you need.

***Linux (GPT) firmware***: The Linux firmware files, including Ubuntu, Buildroot, Debian and other Linux systems, contain `GPT` in the filenames which means that they are using GPT partition scheme. They are referred to as `Linux (GPT) firmware`, which can be compiled in the [Compile Linux firmware(GPT)](linux_compile) chapter.

<a id="flashing-table"></a>

## Flashing Instructions 

***The flashing tools and flashing methods used between different firmware are different. Please follow the table below to flash.***

![](../../../rk3328_img/ROC-RK3328-CC/flashing-table1.png)

## Flashing Tools

Please use the eMMC flashing tools according to your OS:

- To flash to the eMMC:
    + GUI
        * [AndroidTool] (Windows)
    + CLI
        * [upgrade_tool] (Linux)
        * [rkdeveloptool] (Linux)

<font color=#ff0000 size=3>Tip</font>:[AndroidTool] and [upgrade_tool] both support to flashing [Raw Firmware] and [RK Firmware], But [rkdeveloptool] only support to flashing [Raw Firmware].

## AndroidTool

[AndroidTool] is used to flash [Raw Firmware], [RK Firmware] and [Partition Image] to eMMC.

To use [AndroidTool], you need to install [Rockusb Driver] first.

<a id="rockusb-driver"></a>

### Installing Rockusb Driver

Download [DriverAssistant](https://pan.baidu.com/s/1migPY1U#list/path=%2FPublic%2FDevBoard%2FROC-RK3328-CC%2FTools%2FRKTools%2Fwindows&parentPath=%2FPublic%2FDevBoard%2FROC-RK3328-CC
), extract the archive and run `DriverInstall.exe` inside.

![](../../../rk3328_img/ROC-RK3328-CC/started_driverassistant.png)

Click the "驱动安装" button to install the driver. If you want to uninstall the driver, click the "驱动卸载" button.

If your device is in [Rockusb Mode] or [Maskrom Mode], you'll find a `Rockusb Device` in the device manager:

![](../../../rk3328_img/ROC-RK3328-CC/started_driverassistant_dev.png)

### Installing AndroidTool

Flashing tool selection and download:

- Flashing [Raw Firmware]: [AndroidTool_v2.39](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool.7z)
- Flashing RK Android7.1 Firmware: [AndroidTool_v2.38](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool_Release_v2.38.rar)
- Flashing RK Android8.1 Firmware: [AndroidTool_v2.58](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool_Release_V2.58.zip)
- Flashing RK Linux(GPT) Firmware: [AndroidTool_v2.58](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool_Release_V2.58.zip)

Download [AndroidTool], extract it. Locate the file named `config.ini`, and edit it by changing the 4th line from `Selected=1` to `Selected=2`, in order to select English as the default user interface language.

Launch `AndroidTool.exe`:

![](../../../rk3328_img/ROC-RK3328-CC/androidtool.png)

If your device is in [Rockusb Mode], the status line will be "Found One LOADER Device".

If your device is in [Maskrom Mode], the status line will be "Found One MASKROM Device".

### Flashing Raw Firmware

[Raw Firmware] needs to be flashed to offset 0 of eMMC storage. However, in [Rockusb Mode], all LBA writes are offset by 0x2000 sectors. Therefore, the device has to be forced into [Maskrom Mode].

To flash [Raw Firmware] to the eMMC using [AndroidTool], follow the steps below:

1. Force the device into [Maskrom Mode].
2. Run [AndroidTool].
3. Switch to the "Download Image" tab page.
4. Keep the first line of the table unchanged, using the default loader file.
5. Click the right blank cell on the second line, which will pop up a file dialog to open the [Raw Firmware] file.
6. Click the "Run" button to flash.

![](../../../rk3328_img/ROC-RK3328-CC/androidtool_flash_image.png)

### Flashing RK Firmware

To flash [RK Firmware] to the eMMC using [AndroidTool], follow the steps below:

1. Force the device into [Rockusb Mode] or [Maskrom Mode].
2. Run [AndroidTool].
3. Switch to the "Upgrade Firmware" tab page.
4. Click the "Firmware" button, which will pop up a file dialog to open the [RK Firmware] file.
5. The firmware version, loader version and chip info will be read and displayed.
6. Click the "Upgrade" button to flash.

Tips:If you want to flash different system or fail to flash, please read [Flashing Instructions](flash_emmc.html#flashing-table) carefully.

### Flashing Partition Image

Depending on the original firmware of the development board, flashing the partition image to eMMC will be different.

#### [Raw Firmware]

To flash [Partition Image] to the eMMC using [AndroidTool_v2.39], follow the steps below:

1. Force the device into [Rockusb Mode] or [Maskrom Mode].
2. Run [AndroidTool].
3. Switch to the "Download Image" tab page.
4. Keep the first line of the table unchanged.
5. Delete all others unused rows by selecting "Delete Item" from the right-click popup menu.

    ![](../../../rk3328_img/ROC-RK3328-CC/androidtool_del.png)

6. Add partition image to flash by selection "Add Item" from the right-click popup menu.
    + Check on the checkbox on the first cell.
    + Fill in the address with the sector offset (plus `0x2000` if in [Maskrom Mode]) of partition in `parameter.txt` file.
    + Click the right blank cell to browse to the [Partition Image] file.

   ![](../../../rk3328_img/ROC-RK3328-CC/androidtool_add.png)

7. Click the "Run" button to flash.

**Note**:

- You can add multiple partitions to flash by repeating step 6.
- You can skip the partition flashing by checking off the checkbox in front of the address cell.
- In [Maskrom Mode], you must add `0x2000` to the sector offset of the partition in `parameter.txt`. See [Partition Offset](#partition-offset) for more detail.

#### [RK Firmware]

Use `AndroidTool_v2.38` to flashing `RK Android7.1` firmware partitions with default configuration.

Use `AndroidTool_v2.58` to flashing `RK Android8.1` firmware partitions with `rk3328-Android81.cfg` configuration, Right-click the blank section, click `load config`, and select the corresponding configuration file.

Use `AndroidTool_v2.58` to flashing `RK Linux (GPT)` firmware partitions with default configuration.

To flash [Partition Image] to the eMMC using [AndroidTool], follow the steps below:

1. Force the device into [Rockusb Mode].
2. Run [AndroidTool].
3. Switch to the "Download Image" tab page.
4. Make sure the path to the image file is correct. If necessary, click the blank table cell to the right of the path to reselect it.
5. Click the "Run" button to start the upgrade. After the upgrade, the device will restart automatically.

![](../../../rk3328_img/ROC-RK3328-CC/Androidtool58.png)

<a id="upgrade-tool"></a>

## upgrade_tool

[upgrade_tool] is a close-sourced command line tool provided by Rockchip, which supports flashing [Raw Firmware], [RK Firmware] and [Partition Image] to the eMMC.

### Installing upgrade_tool

Flashing tool selection and download:

- Flashing [Raw Firmware] or `Android7.1`: [upgrade_tool_v1.24](http://download.t-firefly.com/product/RK3328/Tools/Linux_Upgrade_Tool/Linux_Upgrade_Tool_v1.24.zip)
- Flashing [RK Firmware] or `Android8.1` : [upgrade_tool_v1.34](http://download.t-firefly.com/product/RK3328/Tools/Linux_Upgrade_Tool/Linux_Upgrade_Tool_1.34.zip)

> [upgrade_tool Download Link](https://gitlab.com/TeeFirefly/RK3328-Nougat/blob/roc-rk3328-cc/RKTools/linux/Linux_Upgrade_Tool/Linux_Upgrade_Tool_v1.24.zip)

Download [upgrade_tool], and install it to your Linux host:

    unzip Linux_Upgrade_Tool_v1.24.zip
    cd Linux_UpgradeTool_v1.24
    sudo mv upgrade_tool /usr/local/bin
    sudo chown root:root /usr/local/bin/upgrade_tool
    sudo chmod 0755 /usr/local/bin/upgrade_tool

Then add `udev` rules by instructions [here](#udev), in order to have permission for the normal user to flash Rockchip devices. If you skip this, you must prefix the following commands with `sudo` to have the right permission.

### Flashing Raw Firmware

[Raw Firmware] needs to be flashed to offset 0 of eMMC storage. However, in [Rockusb Mode], all LBA writes are offset by 0x2000 sectors. Therefore, the device has to be forced into [Maskrom Mode].

To flash [Raw Firmware] to the eMMC using [upgrade_tool], follow the steps below:

1. Force the device into [Maskrom Mode].
2. Run:

    ``` shell
    upgrade_tool db           out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    upgrade_tool wl 0x0       out/system.img
    upgrade_tool rd           # reset device to boot
    ```

Note:

- `rk3328_loader_ddr786_v1.06.243.bin` is the copied loader file after compiling `U-Boot`. It can also be downloaded from [here](https://github.com/rockchip-linux/rkbin/tree/master/rk33) (choose `rk3328_loader_xxx.bin` file).
- `system.img` is [Raw Firmware] after packing, which can also be [Raw Firmware] downloaded from official site (decompress first).

### Flashing RK Firmware

To flash [RK Firmware] to the eMMC using [upgrade_tool], follow the steps below:

1. Force the device into [Rockusb Mode] or [Maskrom Mode].
2. Run:

    ``` shell
    upgrade_tool uf update.img
    ```

if the upgrade failed, please erase the eMMC:

```
upgrade_tool ef update.img
```

### Flashing Partition Image

You can write individual [Partition Image] to the eMMC. Depending on the original content of the eMMC, the instructions can be somewhat different.

**Raw Firmware**

If the original firmware format is raw, chances are that it is using the `GPT` partition scheme, and the predefined offset and size of each partition can be found in `build/partitions.sh` in the SDK. See [Partition Offset](#partition-offset) for more detail.

To flash [Partition Image] to the eMMC using [upgrade_tool], follow the steps below:

1. Force the device into [Maskrom Mode].
2. Use [upgrade_tool] to flash the [Partition Image]:

    ``` shell
    upgrade_tool db         out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    upgrade_tool wl 0x40    out/u-boot/idbloader.img
    upgrade_tool wl 0x4000  out/u-boot/uboot.img
    upgrade_tool wl 0x6000  out/u-boot/trust.img
    upgrade_tool wl 0x8000  out/boot.img
    upgrade_tool wl 0x40000 out/linaro-rootfs.img
    upgrade_tool rd         # reset device to boot
    ```

**RK Firmware**

If the original firmware format is Rockchip, it is using the `parameter` file for partition scheme, and you can use the partition name to flash [Partition Image] directly.

To flash the [Partition Image] to the eMMC using [upgrade_tool], follow the steps below:

1. Force the device into [Rockusb Mode].
2. Use [upgrade_tool] to flash the [Partition Image]:

***Android7.1***

``` shell
upgrade_tool di -b boot.img
upgrade_tool di -k kernel.img
upgrade_tool di -s system.img
upgrade_tool di -r recovery.img
upgrade_tool di -m misc.img
upgrade_tool di resource resource.img
upgrade_tool di -p parameter   # flash parameter
upgrade_tool ul bootloader.bin # flash bootloader
```

***Android8.1***
```
sudo upgrade_tool ul bootloader.bin # flash bootloader
sudo upgrade_tool di -p paramater   # flash parameter
sudo upgrade_tool di -uboot uboot.img
sudo upgrade_tool di -trust trust.img
sudo upgrade_tool di -m misc.img
sudo upgrade_tool di -baseparameter baseparameter.img
sudo upgrade_tool di -b boot.img
sudo upgrade_tool di -k kernel.img
sudo upgrade_tool di -resource resource.img
sudo upgrade_tool di -r recovery.img
sudo upgrade_tool di -s system.img
sudo upgrade_tool di -vendor vendor.img
sudo upgrade_tool di -oem oem.img
```

***Linux(GPT)***

```
upgrade_tool ul $LOADER
upgrade_tool di -p $PARAMETER
upgrade_tool di -uboot $UBOOT
upgrade_tool di -trust $TRUST
upgrade_tool di -b $BOOT
upgrade_tool di -r $RECOVERY
upgrade_tool di -m $MISC
upgrade_tool di -oem $OEM
upgrade_tool di -userdata $USERDATA
upgrade_tool di -rootfs $ROOTFS
```

Note:

- `-b` is a predefined shortcut for `boot` partition. If no shortcuts are available, use partition name instead (`resource` in above example).
- You can customize kernel parameters and partition layout according to [Parameter file format](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf). Once the partition layout is changed, you must flash the `parameter` file first, before reflashing other changed partitions.

### FAQ

If errors occur due to flash storage problem, you can try to low format or erase the flash by:

    upgrade_tool lf   # low format flash
    upgrade_tool ef   # erase flash

## rkdeveloptool

[rkdeveloptool] is an open-source command line flashing tool developed by Rockchip, which is an alternative to the close-source [upgrade_tool].

[rkdeveloptool] **DOES NOT** support proprietary [RK Firmware].

### Installing rkdeveloptool

First, download, compile and install [rkdeveloptool]:

    #install libusb and libudev
    sudo apt-get install pkg-config libusb-1.0 libudev-dev libusb-1.0-0-dev dh-autoreconf
    # clone source and make
    git clone https://github.com/rockchip-linux/rkdeveloptool
    cd rkdeveloptool
    autoreconf -i
    ./configure
    make
    sudo make install

Then add `udev` rules by instructions [here](#udev), in order to have permission for the normal user to flash Rockchip devices. If you skip this, you must prefix the following commands with `sudo` to have the right permission.

### Flashing Raw Firmware

[Raw Firmware] needs to be flashed to offset 0 of eMMC storage. However, in [Rockusb Mode], all LBA writes are offset by 0x2000 sectors. Therefore, the device has to be forced into [Maskrom Mode].

To flash [Raw Firmware] to the eMMC using [rkdeveloptool], follow the steps below:

1. Force the device into [Maskrom Mode].
2. Run:

    ``` shell
    rkdeveloptool db           out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    rkdeveloptool wl 0x0       out/system.img
    rkdeveloptool rd           # reset device to boot
    ```

Note:

- `rk3328_loader_ddr786_v1.06.243.bin` is the copied loader file after compiling `U-Boot`. It can also be downloaded from [here](https://github.com/rockchip-linux/rkbin/tree/master/rk33) (choose `rk3328_loader_xxx.bin` file).
- `system.img` is [Raw Firmware] after packing, which can also be the [Raw Firmware] downloaded from official site (decompress it first).

### Flashing Partition Image

The following instructions **ONLY APPLIY** to boards which are flashed with [Raw Firmware] and use `GPT` partition scheme. The predefined offset and size of each partition can be found in `build/partitions.sh` in the SDK. See [Partition Offset](#partition-offset) for more detail.

To flash [Partition Image] to the eMMC using [rkdeveloptool], follow the steps below:

1. Force the device into [Maskrom Mode].
2. Run:

    ``` shell
    rkdeveloptool db           out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    rkdeveloptool wl 0x40      out/u-boot/idbloader.img
    rkdeveloptool wl 0x4000    out/u-boot/uboot.img
    rkdeveloptool wl 0x6000    out/u-boot/trust.img
    rkdeveloptool wl 0x8000    out/boot.img
    rkdeveloptool wl 0x40000   out/linaro-rootfs.img
    rkdeveloptool rd           # reset device to boot
    ```

## udev

Create `/etc/udev/rules.d/99-rk-rockusb.rules` with following content[1](https://github.com/rockchip-linux/rkdeveloptool/blob/master/99-rk-rockusb.rules). Replace the group `users` with your actual Linux group if neccessary:

``` shell
SUBSYSTEM!="usb", GOTO="end_rules"

# RK3036
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="301a", MODE="0666", GROUP="users"
# RK3229
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="320b", MODE="0666", GROUP="users"
# RK3288
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="320a", MODE="0666", GROUP="users"
# RK3328
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="320c", MODE="0666", GROUP="users"
# RK3368
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="330a", MODE="0666", GROUP="users"
# RK3399
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="330c", MODE="0666", GROUP="users"

LABEL="end_rules"
```

Reload the udev rules to take effect without reboot:

    sudo udevadm control --reload-rules
    sudo udevadm trigger

<a id="partition-offset"></a>

## Partition Offset

### GPT Partition

The offset of partition image can be obained by following command(assuming you are in the directory of Firefly Linux SDK):

    (. build/partitions.sh ; set | grep _START | \
    while read line; do start=${line%=*}; \
    printf "%-10s 0x%08x\n" ${start%_START*} ${!start}; done )

which gives result of:

    ATF        0x00006000
    BOOT       0x00008000
    LOADER1    0x00000040
    LOADER2    0x00004000
    RESERVED1  0x00001f80
    RESERVED2  0x00002000
    ROOTFS     0x00040000
    SYSTEM     0x00000000

### parameter

If [RK Firmware] is used, `parameter.txt` is used to define partition layout.

Here's a handy script to list the partition offsets in `parameter.txt`:

``` shell
#!/bin/sh

PARAMETER_FILE="$1"
[ -f "$PARAMETER_FILE" ] || { echo "Usage: $0 <parameter_file>"; exit 1; }

show_table() {
    echo "$1"
    echo "--------"
    printf "%-20s %-10s %s\n" "NAME" "OFFSET" "LENGTH"
    for PARTITION in `cat ${PARAMETER_FILE} | grep '^CMDLINE' | sed 's/ //g' | sed 's/.*:\(0x.*[^)])\).*/\1/' | sed 's/,/ /g'`; do
        NAME=`echo ${PARTITION} | sed 's/\(.*\)(\(.*\))/\2/'`
        START=`echo ${PARTITION} | sed 's/.*@\(.*\)(.*)/\1/'`
        LENGTH=`echo ${PARTITION} | sed 's/\(.*\)@.*/\1/'`
        START=$((START + $2))
        printf "%-20s 0x%08x %s\n" $NAME $START $LENGTH
    done
}

show_table "Rockusb Mode" 0
echo
show_table "Maskrom Mode" 0x2000
```

Save it as a script in `/usr/local/bin/show_rk_parameter.sh` and give the script executing permission.

Here's an example of showing partition offsets defined in `RK3328 Android SDK`:

```text
$ show_rk_parameter.sh device/rockchip/rk3328/parameter.txt
Rockusb Mode
--------
NAME                 OFFSET     LENGTH
uboot                0x00002000 0x00002000
trust                0x00004000 0x00004000
misc                 0x00008000 0x00002000
baseparamer          0x0000a000 0x00000800
resource             0x0000a800 0x00007800
kernel               0x00012000 0x00010000
boot                 0x00022000 0x00010000
recovery             0x00032000 0x00010000
backup               0x00042000 0x00020000
cache                0x00062000 0x00040000
metadata             0x000a2000 0x00008000
kpanic               0x000aa000 0x00002000
system               0x000ac000 0x00300000
userdata             0x003ac000 -

Maskrom Mode
--------
NAME                 OFFSET     LENGTH
uboot                0x00004000 0x00002000
trust                0x00006000 0x00004000
misc                 0x0000a000 0x00002000
baseparamer          0x0000c000 0x00000800
resource             0x0000c800 0x00007800
kernel               0x00014000 0x00010000
boot                 0x00024000 0x00010000
recovery             0x00034000 0x00010000
backup               0x00044000 0x00020000
cache                0x00064000 0x00040000
metadata             0x000a4000 0x00008000
kpanic               0x000ac000 0x00002000
system               0x000ae000 0x00300000
userdata             0x003ae000 -
```

[Getting Started]: getting_started.md
[FAQ]: faqs.md
[Serial Debug]: debug.md
[Building Linux Root Filesystem]: linux_build_ubuntu_rootfs.md
[Contact]: resource.md#community
[Raw Firmware]: getting_started.md#raw-firmware-format
[RK Firmware]: getting_started.md#rk-firmware-format
[Partition Image]: getting_started.md#partition-image
[SDCard Installer]: flash_sd.md#sdcard-installer
[Etcher]: flash_sd.md#etcher
[dd]: flash_sd.md#dd
[SD Firmware Tool]: flash_sd.md#sd-firmware-tool
[AndroidTool]: flash_emmc.md#androidtool
[upgrade_tool]: flash_emmc.md#upgrade-tool
[rkdeveloptool]: flash_emmc.md#rkdeveloptool
[Rockusb Mode]: flash_emmc.md#rockusb-mode
[Maskrom Mode]: flash_emmc.md#maskrom-mode
[Rockusb Driver]: flash_emmc.md#rockusb-driver
[ROC-RK3328-CC]: http://en.t-firefly.com/product/rocrk3328cc.html "ROC-RK3328-CC Official Website"
[Download Page]: http://en.t-firefly.com/doc/download/34.html
[Forum]: http://bbs.t-firefly.com/
[Facebook]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[Youtube]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[Twitter]: https://twitter.com/TeeFirefly
[Shop]: http://shop.t-firefly.com/
[USB Serial Adapter]: https://www.firefly.store/products/usb-to-uart-module-cp2104 
[5V2A US Adapter]: https://www.firefly.store/products/5v2a-us-adapter-3c-fcc-ce 
[eMMC Flash]: https://www.firefly.store/products
[Storage Map]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map