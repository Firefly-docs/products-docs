# Boot Mode description

## Introduction

RK3128 has flexible boot modes. Under normal circumstances, unless hardware is damaged, FirePrime development board will never become brick (brick means not able to boot or flash).  

If accident happens during upgrading firmware, bootloader is broken, making it impossible to upgrade again. The device can enter into MaskRom mode as the last resort.

## Loading mode

RK3128 has 16KB BootRom and 8KB internal SRAM, which supports booting from following devices:

* 8bit async Nand Flash  
* 8bit toggle Nand Flash  
* SPI interface  
* eMMC interface  
* SDMMC interface

In addition, the RK3128 supports downloading the firmware from the OTG interface.

## Boot order
The boot order is:  

* Soc powers up and initializes.
* BootRom code runs in SRAM, loads and verifies bootloader's bootstrap code from storage device.
* If the verification passes, run the bootloader bootstrap code.
* The bootloader initializes DDR RAM, loads the complete bootloader into DDR RAM and runs it.
* Bootloader loads Linux kernel from storage device, then executes it.
* Linux kernel takes control of everything now.

## Boot Mode

There are three boot modes for the RK3128:   

* Normal mode
* Loader mode
* MaskRom mode

### Normal Mode

Normal mode is the normal startup process, and each component is loaded in order to enter the system normally.

### Loader Mode

In Loader mode, bootloader will enter into upgrade state, waiting for commands from host, which is used in firmware upgrading.  

To enter Loader mode, make the bootloader aware that the RECOVERY key is pressed and USB cable is connected:    

* The first one, device is cut off all the power sources, such as power adapter and Micro USB OTG connection:  
  * Keep micro USB OTG cable connected with host PC.
  * Press and hold RECOVERY key.
  * Connect micro USB OTG cable to the device.
  * After around two seconds, release RECOVERY key.

* The other way:  
  * Use micro USB OTG cable to connect host and device together.
  * Press and hold RECOVERY key.
  * Shortly press RESET key.
  * After around two seconds, release RECOVERY key.

### MaskRom Mode

The MaskRom mode is used for system recovery when the bootloader is damaged.

In general, the BootRom code will not enter the MaskRom mode, and only if bootloader validation fails (the IDR is not readable or the bootloader is damaged), the BootRom code will enter the MaskRom mode . At this time, the BootRom code waits for the host to pass the bootloader code through the USB interface, and load and run the code.

If you want to forcibly enter the MaskRom mode, please refer to the chapter [《MaskRom》](maskrom_mode.md).