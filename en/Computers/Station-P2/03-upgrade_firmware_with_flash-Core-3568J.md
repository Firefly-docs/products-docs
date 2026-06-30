## Maskrom Mode
If you enter Maskrom mode through ["How to enter MaskRom mode"](04-maskrom_mode.md), and the board has both Nor-Flash and EMMC storage media, this raises the question of which storage media we use to burn firmware.
The following introduces how to download firmware to different storage media. 

### Download to Nor-Flash
In Maskrom mode, the system will download firmware to Nor-Flash **by default** if the board has the Nor-Flash. However, Nor-Flash storage space is too small to load the whole system firmware, so only small files will be load, such as `MiniLoaderAll.bin` . 

If we accidentally download the whole system firmware to Nor-Flash, the board will turn brick due to the failure of downloading firmware(`下载固件失败`). At this time, if we want to re-enter Maskrom mode, we could  short circuit test points(or pins)  between  D0(CLK) and GND of Nor-Flash, refer to chapter [MaskRom mode](04-maskrom_mode.md) for more details. 

### Download to EMMC
To download firmware to EMMC, we need to download Boot first, and then select to switch storage media to EMMC. The specific operation steps are as follows: 

**Step 1:** We need to get `MiniLoaderAll.bin` which will download to the Boot, this kind * Loader *. bin can be compiled firmware of the corresponding device model in SDK to generate, we could find it in the directory `rockdev` finally. 
```shell
#Android SDK
rockdev//MiniLoaderAll.bin

#Linux SDK
rockdev/MiniLoaderAll.bin
```

If we have not compiled, we can download a [MiniLoaderAll.bin](https://www.t-firefly.com/share/index/index/id/4e86cc234c3b26e2b12084e383074ada.html) for the corresponding CPU model.

**Step 2:** Click the option `Adanced Function` of the RKDevTool found at directory `RKTools/windows/AndroidTool` in SDK, and then download the file `MiniLoaderAll.bin` to the Boot by clicking the button `Download` when `MiniLoaderAll.bin` has been selected at the Boot text box. The figure shown as following:

![](../../../rk356x_img/Core-3568J/load_emmc_with_flash_en01.png)

**Step 3:** Click the button `List Storage`to get the storage media list, then we select the storage media `Emmc` of the list.

![](../../../rk356x_img/Core-3568J/load_emmc_with_flash_en02.png)

* ` X ` means device does not exist
* ` 0 ` means the media exist, but is not selected
* ` √` means the media exist and in the selected state

**Step 4:** Click the button `Switch Storage`, the storage media `Emmc` state of the list  will go from `0` to `√`, indicating EMMC is selected. At this time, the firmware we dowload to the board will be burned into EMMC.

![](../../../rk356x_img/Core-3568J/load_emmc_with_flash_en03.png)

**Step 5:** For sure the whole system firmware will boot start at EMMC, we should click the button `EraseAll` to clear all the storage media.

![](../../../rk356x_img/Core-3568J/load_emmc_with_flash_en04.png)

**Step 6:** Click the option `Upgrade Firmware`，select one system firmware we want to download in and click `Upgrade` done

![](../../../rk356x_img/Core-3568J/load_emmc_with_flash_en05.png)


