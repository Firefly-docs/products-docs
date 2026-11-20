# FAQ

<Accordions>

<Accordion title="Q: What firmware upgrade methods does the EC-A1684XJD4 FD support?" type="single" id="bm1684_upgrade_faq_1">

**A:** The EC-A1684XJD4 FD upgrades the firmware via a TF card:

* The TF card must use an MBR partition table and be formatted as FAT32;
* Extract all files from the firmware package to the root directory of the TF card (no folders on the TF card);
* Insert the TF card into the card slot and power on the device, the upgrade will complete automatically.

For EC series computers, see [Firmware upgrade](../../fw_upgrade.md); for AIBOX series, see [System firmware upgrade](fw-upgrade-by-sdcard.md).

</Accordion>

<Accordion title="Q: How to tell the upgrade status during the upgrade?" type="single" id="bm1684_upgrade_faq_2">

**A:** Check the on-board LED indicators:

1. During the upgrade, the LEDs flash briefly, which means the upgrade is in progress.
2. If the upgrade is successful, the green LED keeps flashing.
3. If the upgrade fails, all LEDs go out.

</Accordion>

<Accordion title="Q: What to do if the TF card cannot be formatted as FAT32?" type="single" id="bm1684_upgrade_faq_3">

**A:** If the TF card is larger than 32GB, it may not be able to be formatted as FAT32 due to Windows system limitations. It is recommended to choose a TF card of 32GB or less.

</Accordion>

<Accordion title="Q: How long does the upgrade take? What to do if the upgrade fails?" type="single" id="bm1684_upgrade_faq_4">

**A:** The upgrade takes about six minutes, please be patient. If the upgrade fails, please make sure the TF card uses an MBR partition and FAT32 format, and all files in the firmware package have been extracted directly to the root directory of the TF card, then power on and upgrade again.

</Accordion>

<Accordion title="Q: How do I upgrade or downgrade between the new and old system firmware?" type="single" id="bm1684_upgrade_faq_5">

**A:** Only BM1684X machines involve an MCU firmware update: SOC firmware 20230501 and 20230301/20221201 are two different generations. Before upgrading or downgrading between them, the MCU firmware must be upgraded or downgraded first. Obtain firmware packages from the [Download Center](https://community.t-firefly.com/en/doc/download/208).

1. Check whether the MCU firmware is old or new: run the following command on the device. If the result is `"board type": "0x20"`, it is the old firmware:

```
cat /sys/bus/i2c/devices/1-0017/information | grep "board type"
```

2. Upgrade the MCU firmware (required when upgrading to the 20230501 firmware): download `sophgo-bsp-mcu-se7_20230906_arm64.deb`, upload it to the device, and run:

```
sudo dpkg -i sophgo-bsp-mcu-se7_20230906_arm64.deb
```
3. Downgrade the MCU firmware (required when downgrading to the 20230301/20221201 firmware), choose either method:
   * Method 1: download `sophgo-bsp-mcu-sm7mini_20221111_arm64.deb`, upload it to the device, and run `sudo dpkg -i sophgo-bsp-mcu-sm7mini_20221111_arm64.deb`;
   * Method 2 (for BSP developers): flash `STM32L071C8-mcu-v5-2022-11-11-10-24-11.bin` with a dedicated programmer.
4. Upgrade or downgrade the SOC firmware: follow [Upgrading System Firmware with a TF Card](fw-upgrade-by-sdcard.md) to prepare a TF upgrade card.

</Accordion>

</Accordions>
