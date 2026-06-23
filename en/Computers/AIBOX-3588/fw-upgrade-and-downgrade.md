# Upgrade and downgrade between old and new firmware

WARNING:
- For 1684X machines only! (There's no need for 1684)
- Upgrading/downgrading MCU firmware before upgrading/downgrading SOC firmware

If your AIO-1684X-JD4 (SE7) or ICORE-1684XQ (SM7) has the firmware version of 20230501, and you need to downgrade it to a lower 20230301 or 20221201, then you must downgrade MCU firmware first, whereas you must upgrade MCU firmware first in the reverse case. You can download the MCU firmware and SOC firmware from the [Download Center].

## How to tell if the MCU firmware is new or old?

Run the following command on the board:

```
cat /sys/bus/i2c/devices/1-0017/information  | grep "board type"
```

If it turns out to be 0x20:

```
        "board type": "0x20",
```

then this board is equipped with an old MCU firmware.

## Upgrading MCU firmware

Download `sophgo-bsp-mcu-se7_20230906_arm64.deb` and upload it to the board. Then run the following command on the board:

    sudo dpkg -i sophgo-bsp-mcu-se7_20230906_arm64.Double

## Downgrading MCU firmware

### Method 1: install the MCU deb file

This applies to normal user.

Download `sophgo-bsp-mcu-sm7mini_20221111_arm64.deb`, and upload it to the board. Run on the board:

    sudo dpkg -i sophgo-bsp-mcu-sm7mini_20221111_arm64.deb

### Method 2: use a flasher

This applies to BSP developer, with specialized flasher.

Download `STM32L071C8-mcu-v5-2022-11-11-10-24-11.bin`, and update with the flasher.

## Upgrading or Downgrading SOC firmware

To upgrade, you need to download the `Public-1684-Ubuntu2004-Sdk230501-Build20230915.zip`, To downgrade, you need to download the `EC-A1684XJD4-Ubuntu2004-20230317.zip`, and follow instructions in: [Use TF card to upgrade firmware](fw-upgrade-by-sdcard.md)

[Download Center]: https://en.t-firefly.com/doc/download/292.html#other_705