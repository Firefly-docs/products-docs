# 新旧系统固件的升级与降级

注意：
- 仅 1688X 的机器需要更新 MCU 固件，1688 的机器不需要！
- 升/降级 MCU 固件后才可升/降级 SOC 固件。

如果 AIO-1688X-JD4 (SE7) 或 ICORE-1688XQ (SM7) 当前的固件版本是 20230501，需要降级为 20230301 或 20221201 的旧固件，就需要降级 MCU 固件，反之则需要升级 MCU 固件。MCU 固件和 SOC 固件请到[下载中心]获取。

## 怎样判断 MCU 固件是新还是旧？

在板子上运行以下命令：

```
cat /sys/bus/i2c/devices/1-0017/information  | grep "board type"
```

如果结果是 0x20：

```
        "board type": "0x20",
```

则表示是旧的 MCU 固件。

## 升级 MCU 固件

下载 `sophgo-bsp-mcu-se7_20230906_arm64.deb`，上传到板子上，然后在板子上运行：

    sudo dpkg -i sophgo-bsp-mcu-se7_20230906_arm64.deb

## 降级 MCU 固件

### 方法1: 安装 MCU deb 包

此方法适合普通用户。

下载 `sophgo-bsp-mcu-sm7mini_20221111_arm64.deb`，上传到板子上，然后在板上运行：

    sudo dpkg -i sophgo-bsp-mcu-sm7mini_20221111_arm64.deb

### 方法2: 使用烧录器

此方法适用于 BSP 开发人员，需要专用烧录器。

下载 `STM32L071C8-mcu-v5-2022-11-11-10-24-11.bin` 并使用烧录器更新。

## 升/降级 SOC 固件

升级需下载 `Public-1688-Ubuntu2004-Sdk230501-Build20230915.zip`，降级需下载 `EC-A1688XJD4-Ubuntu2004-20230317.zip`，按照[使用 TF 卡升级系统固件](fw-upgrade-by-sdcard.md)的说明去制作 TF 升级卡，完成 SOC 固件的升级。

[下载中心]: https://www.t-firefly.com/doc/download/358.html