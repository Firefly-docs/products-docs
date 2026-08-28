# MaskRom 模式

RK182X 开发套件不支持 Loader 模式。通过 USB 升级固件或恢复 bootloader 时，必须使用主板上的 `MaskRom` 按键。

**请仔细阅读以下步骤，并在断电状态下操作。**

## 进入 MaskRom 模式

1. 断开开发套件电源。
2. 将拨码开关 `USB SEL` 切换到 `1`。
3. 使用 Type-A 数据线连接 OTG 接口和主机电脑。
4. 按住主板上的 `MaskRom` 按键。
5. 保持按键按下，同时给开发套件上电。
6. 使用烧写工具检查是否发现 MaskRom 设备。
7. 设备被识别后，松开按键。

此时主板已经准备好烧写固件。

![](../../../gs1-n2_img/common/upgrade_maskrom_zh.png)

## 检查 MaskRom 模式

### Windows

AndroidTool 的设备列表中应显示 MaskRom 设备。如果没有识别，请检查 USB 驱动、Type-A 数据线、`USB SEL` 拨码开关和 OTG 连接。

### Linux

运行 `upgrade_tool`，检查连接设备是否显示为 MaskRom：

```shell
sudo upgrade_tool
```
