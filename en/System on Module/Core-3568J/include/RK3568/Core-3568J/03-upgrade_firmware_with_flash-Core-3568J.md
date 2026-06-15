## Maskrom Mode
If you enter Maskrom mode through ["How to enter MaskRom mode"](04-maskrom_mode.md), and the board has both Nor-Flash and EMMC storage media, this raises the question of which storage media we use to burn firmware.
The following introduces how to download firmware to different storage media. 

### Download to Nor-Flash
In Maskrom mode, the system will download firmware to Nor-Flash **by default** if the board has the Nor-Flash. However, Nor-Flash storage space is too small to load the whole system firmware, so only small files will be load, such as `MiniLoaderAll.bin` . 

If we accidentally download the whole system firmware to Nor-Flash, the board will turn brick due to the failure of downloading firmware(`下载固件失败`). At this time, if we want to re-enter Maskrom mode, we could  short circuit test points(or pins)  between  D0(CLK) and GND of Nor-Flash, refer to chapter [MaskRom mode](04-maskrom_mode.md) for more details. 

### Download to EMMC
There are two methods to burn firmware into EMMC. One is the burning method provided by Rockchip, which needs to burn `MiniLoaderAll.bin` and switch storage media. 
The other is a reference burning method provided by Firefly to facilitate everyone to burn, this method does not need to switch the memory storage media in operation, and it can also support uprade_tool for Linux System.

#### The 1nd Method From Firefly

<font color="red">This method is only available in the latest official SDK compilation firmware or the latest official firmware.</font> If the firmware is accidentally downloaded to the NOR flash in Maskrom mode, causing the device to fail to start up properly after reboot, you can enter the [MaskRom mode](04-maskrom_mode.md) and upgrade the latest SDK compilation or the firmware provided by the official. Regardless of whether the upgrading is successful or not, when the machine restarts, if there is data in the NOR flash, it will be automatically erased. The erasing process takes about 30 to 60 seconds. After successful erasing, the machine will automatically enter the Loader mode, and then you can directly [upgrade the firmware](03-upgrade_firmware.html#upgrade-the-firmware).

#### The 2rd Method From Rockchip
To download firmware to EMMC, we need to download Boot first, and then select to switch storage media to EMMC. The specific operation steps are as follows: 

**Step 1:** We need to get `MiniLoaderAll.bin` which will download to the Boot, this kind * Loader *. bin can be compiled firmware of the corresponding device model in SDK to generate, we could find it in the directory `rockdev` finally. 
```shell
#Android SDK
rockdev/Image-rk3568_firefly_aioj/MiniLoaderAll.bin

#Linux SDK
rockdev/MiniLoaderAll.bin
```

If we have not compiled, we can download a [MiniLoaderAll.bin](https://www.t-firefly.com/share/index/index/id/4e86cc234c3b26e2b12084e383074ada.html) for the corresponding CPU model.

**Step 2:** Click the option `Adanced Function` of the RKDevTool found at directory `RKTools/windows/AndroidTool` in SDK, and then download the file `MiniLoaderAll.bin` to the Boot by clicking the button `Download` when `MiniLoaderAll.bin` has been selected at the Boot text box. The figure shown as following:

![](../../img/Core-3568J/load_emmc_with_flash01.png)

**Step 3:** Click the button `List Storage`to get the storage media list, the `SPINOR` is selected in the list, we click `EraseAll` to erase Nor-Flash

![](../../img/Core-3568J/maskrom_erease_spinor_flash.png)

* ` X ` means device does not exist
* ` 0 ` means the media exist, but is not selected
* ` √` means the media exist and in the selected state

**Step 4:** Select the storage media `Emmc` of the list and click the button `Switch Storage`

![](../../img/Core-3568J/load_emmc_with_flash02.png)

the storage media `Emmc` state of the list  will go from `0` to `√`, indicating EMMC is selected. At this time, the firmware we dowload to the board will be burned into EMMC

![](../../img/Core-3568J/load_emmc_with_flash03.png)

**Step 5:** click the button `EraseAll` to erase EMMC

![](../../img/Core-3568J/load_emmc_with_flash04.png)

**Step 6:** Click the option `Upgrade Firmware`，select one system firmware we want to download in and click `Upgrade` done

![](../../img/Core-3568J/load_emmc_with_flash05.png)


