# MaskRom Mode

## Introduction

MaskRom mode is the bottom line preventing the device from bricking. Enforcing device into MaskRom mode involves hardware operation, which is risky. Therefore, please try to put the device into Loader mode, or boot the device with sd-card, before risking MaskRom mode.  

**<font color=#ff0000 size=3>Please read and operate with great care!</font>**

## Firefly-RK3128 enter MaskRom mode

principle：  
Artificial to the Flash data pin connected to ground, the system will think Flash data error, so clearing the Flash data.

1. Power down the device.
2. Plug out SD card.
3. Use a Micro USB OTG cable to connnect device and host pc.
4. Use metal tweezers to turn on the two test points shown on the red board on the core board as shown below.
5. Power on the board.
6. Wait a moment, then release the metal tweezers.

![](../../../rk3128_img/Core-3128J/maskrom_1.png)

Device should enter MaskRom mode:

![](../../../rk3128_img/Core-3128J/win_3128_tool_maskrom.png)