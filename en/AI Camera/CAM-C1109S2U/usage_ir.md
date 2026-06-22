# IR

CAM-C1109S2U is equipped with an IR infrared mainboard. And it is equipped with a photosensitive sensor to identify daytime scenes and nighttime scenes. The following uses IPC firmware for testing.

Test method:

1. Night scene: Cover the photosensitive sensor with your finger to simulate a night scene. The infrared light of the IR infrared mainboard will automatically turn on. The covered part is shown in the figure below:

   ![](../../../rv1126_img/CAM-C1109S2U/ir.png)

2. Daytime scene: Release your finger to cover the photosensitive sensor to simulate a daytime scene. The infrared light of the IR infrared mainboard will automatically turn off.

Description:

When the brightness of the light received by the photosensitive sensor is low, the hardware will automatically turn on the infrared light of the IR infrared mainboard and switch the camera from color mode to black and white mode.

When the brightness of the light received by the photosensitive sensor is high, the hardware will automatically turn off the infrared light of the IR infrared mainboard and switch the camera from black and white mode to color mode.

Note:

The function of the camera automatically switching from color mode to black and white mode is not applicable to Debian firmware. If necessary, please adapt it yourself according to the provided code.

* IRCUT (filter switching) automatic switch

Enter the IP address in the PC browser. Click [Configuration-->System-->System Settings] to enter the configuration interface. Then turn IRCUT off and on in the marked area of ​​the picture. 0 means turn off IRCUT. 1 means turn on IRCUT.

![](../../../rv1126_img/CAM-C1109S2U/ircut.png)

## Code directory

rkaiq_switch_mode

* Path: /path/to/sdk/app/rkaiq_switch_mode

* Description: The script and program switch between the camera color mode and black and white mode according to the brightness of the photosensor, and complete the IRCUT. If you need to use it on Debian firmware, please modify the Makefile compiler path on the Debian system and recompile it.