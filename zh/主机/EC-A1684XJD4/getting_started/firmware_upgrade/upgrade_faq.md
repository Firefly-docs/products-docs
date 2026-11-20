# FAQ

<Accordions>

<Accordion title="Q：EC-A1684XJD4 支持哪些固件升级方式？" type="single" id="bm1684_upgrade_faq_1">

**A：** EC-A1684XJD4 通过 TF 卡升级固件：

* TF 卡使用 MBR 分区，并格式化为 FAT32 格式；
* 将待升级固件压缩包内的全部文件解压到 TF 卡根目录（TF 卡内无文件夹）；
* TF 卡插入卡槽后上电开机，设备自动完成升级。

EC 系列整机详细步骤请参阅[《固件升级》](../../fw_upgrade.md)，AIBOX 系列请参阅[《系统固件升级》](fw-upgrade-by-sdcard.md)。

</Accordion>

<Accordion title="Q：升级过程中如何判断升级状态？" type="single" id="bm1684_upgrade_faq_2">

**A：** 通过板载 LED 指示灯判断：

1. 升级过程中，LED 灯会短暂闪烁，表示升级正在进行。
2. 如果升级成功，绿色 LED 灯会一直闪烁。
3. 如果升级失败，所有 LED 灯都会灭掉。

</Accordion>

<Accordion title="Q：TF 卡无法格式化成 FAT32 怎么办？" type="single" id="bm1684_upgrade_faq_3">

**A：** TF 卡如果超过 32GB，可能因 Windows 系统的限制而无法格式化成 FAT32 格式，建议选择 32GB 以下大小的 TF 卡。

</Accordion>

<Accordion title="Q：升级大约需要多长时间？升级失败怎么办？" type="single" id="bm1684_upgrade_faq_4">

**A：** 升级时间有点长，大概要六分钟左右，请耐心等待。若升级失败，请确认 TF 卡为 MBR 分区 + FAT32 格式、固件包内全部文件已直接解压到 TF 卡根目录，然后重新上电升级。

</Accordion>

<Accordion title="Q：如何在新旧系统固件之间升级与降级？" type="single" id="bm1684_upgrade_faq_5">

**A：** 仅 BM1684X 机器涉及 MCU 固件更新：SOC 固件 20230501 与 20230301/20221201 互为新旧两代，两者互相升/降级前必须先升/降级 MCU 固件。固件包请到[下载中心](https://community.t-firefly.com/doc/download/225)获取。

1. 判断 MCU 固件新旧：在设备上运行以下命令，若结果为 `"board type": "0x20"` 则为旧固件：

```
cat /sys/bus/i2c/devices/1-0017/information | grep "board type"
```

2. 升级 MCU 固件（升级到 20230501 新固件必做）：下载 `sophgo-bsp-mcu-se7_20230906_arm64.deb` 上传到设备，执行：

```
sudo dpkg -i sophgo-bsp-mcu-se7_20230906_arm64.deb
```
3. 降级 MCU 固件（降级到 20230301/20221201 旧固件必做），两种方法任选：
   * 方法一：下载 `sophgo-bsp-mcu-sm7mini_20221111_arm64.deb` 上传到设备，执行 `sudo dpkg -i sophgo-bsp-mcu-sm7mini_20221111_arm64.deb`；
   * 方法二（BSP 开发人员）：使用专用烧录器烧录 `STM32L071C8-mcu-v5-2022-11-11-10-24-11.bin`。
4. 升/降级 SOC 固件：按照[使用 TF 卡升级系统固件](fw-upgrade-by-sdcard.md)制作 TF 升级卡完成升/降级。

</Accordion>

</Accordions>
