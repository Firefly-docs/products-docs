# Camera 使用

## 简介
AIO-3399J 开发板分别带有两个 MIPI，MIPI 支持最高 4K 拍照，并支持 1080P 30fps 以上视频录制。此外，开发板还支持 USB 摄像头。


本文以 OV13850/OV5640 摄像头为例，讲解在该开发板上的配置过程。

* 接口效果图

![](../../../rk3399_img/AIO-3399J/camera_interface.jpg)

## DTS 配置

```
isp0: isp@ff910000 {
    …
    status = "okay";
}
isp1: isp@ff920000 {
    …
    status = "okay";
}
```

## 驱动说明

与摄像头相关的代码目录如下：

```
Android：
 `- hardware/rockchip/camera/
    |- CameraHal             // 摄像头的 HAL 源码
    `- SiliconImage          // ISP 库，包括所有支持模组的驱动源码
       `- isi/drv/OV13850    // OV13850 模组的驱动源码
          `- calib/OV13850.xml // OV13850 模组的调校参数
 `- device/rockchip/rk3399/
    |- rk3399_firefly_aio_box
    |  `- cam_board.xml      // 摄像头的参数设置

 Kernel：
 |- kernel/drivers/media/video/rk_camsys  // CamSys 驱动源码
 `- kernel/include/media/camsys_head.h
```

## 配置原理

设置摄像头相关的引脚和时钟，即可完成配置过程。

从以下摄像头接口原理图可知，需要配置的引脚有：`AF_VDD28`、`DOVDD18`、`AVDD28`、`DVDD12`、`PWDN1`、`RST` 和 `MCLK`。

* MIPI 接口

![](../../../rk3399_img/AIO-3399J/camera_mipi_interface.jpg)

* `AF_VDD28` 可不做配置。
* `DOVDD18`、`AVDD28` 由 `DVP_PWR` 控制，`DVP_PWR` 对应 RK3399 的 `GPIO1_C1`：

![](../../../rk3399_img/AIO-3399J/camera_dvp_pwr.jpg)

* `DVDD12` 由 `CIF_POWER` 引脚控制，`CIF_POWER` 对应 RK3399 上的 `GPIO1_A1` 引脚：

![](../../../rk3399_img/AIO-3399J/camera_cif_pwr.jpg)

* `MIPI CIF`：PWDN0(共用)、PWDN1、RST 对应 GPIO1_C1、GPIO1_A1、GPIO0_B0 引脚：

![](../../../rk3399_img/AIO-3399J/camera_mipi_cif.jpg)

在开发板中，除了 `DVDD12 (CIF_POWER)` 要在 DTS 中设置以外，其它引脚都是在 `cam_board.xml` 中设置。
## 配置步骤

### 配置 Android

修改 `device/rockchip/rk3399/$(TARGET_PRODUCT)/cam_board.xml` 来注册摄像头：

```
<BoardFile>
<BoardXmlVersion version="v0.0xf.0"></BoardXmlVersion>
<CamDevie>
<HardWareInfo>
<Sensor>
<SensorName name="OV13850"/>
<SensorLens name="50013A1"/>
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_1B"/>
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN"/>
<SensorI2cBusNum busnum="1"/>
<SensorI2cAddrByte byte="2"/>
<SensorI2cRate rate="100000"/>
<SensorAvdd name="NC" min="28000000" max="28000000" delay="0"/>
<SensorDvdd name="NC" min="12000000" max="12000000" delay="0"/>
<SensorDovdd name="NC" min="18000000" max="18000000" delay="5000"/>
<SensorMclk mclk="24000000" delay="1000"/>
<SensorGpioPwen ioname="RK30_PIN1_PC1" active="1" delay="1000"/>
<SensorGpioRst ioname="RK30_PIN0_PB0" active="0" delay="1000"/>
<SensorGpioPwdn ioname="RK30_PIN1_PA1" active="0" delay="0"/>
<SensorFacing facing="back"/>
<SensorInterface interface=" MIPI "/>
<SensorMirrorFlip mirror="0"/>
<SensorOrientation orientation="180"/>
<SensorPowerupSequence seq="1234"/>
<SensorFovParemeter h="60.0" v="60.0"/>
<SensorAWB_Frame_Skip fps="15"/>
<SensorPhy phyMode="CamSys_Phy_Mipi" lane="2" phyIndex="1" sensorFmt="CamSys_Fmt_Raw_10b"/>
</Sensor>
<VCM>
<VCMDrvName name="DW9714"/>
<VCMName name="HuaYong6505"/>
<VCMI2cBusNum busnum="1"/>
<VCMI2cAddrByte byte="0"/>
<VCMI2cRate rate="0"/>
<VCMVdd name="NC" min="0" max="0" delay="0"/>
<VCMGpioPower ioname="NC" active="0" delay="1000"/>
<VCMGpioPwdn ioname="NC" active="0" delay="0"/>
<VCMCurrent start="20" rated="80" vcmmax="100" stepmode="13" drivermax="100"/>
</VCM>
<Flash>
<FlashName name="Internal"/>
<FlashI2cBusNum busnum="0"/>
<FlashI2cAddrByte byte="0"/>
<FlashI2cRate rate="0"/>
<FlashTrigger ioname="NC" active="0"/>
<FlashEn ioname="NC" active="0"/>
<FlashModeType mode="1"/>
<FlashLuminance luminance="0"/>
<FlashColorTemp colortemp="0"/>
</Flash>
</HardWareInfo>
<SoftWareInfo>
<AWB>
<AWB_Auto support="1"/>
<AWB_Incandescent support="1"/>
<AWB_Fluorescent support="1"/>
<AWB_Warm_Fluorescent support="1"/>
<AWB_Daylight support="1"/>
<AWB_Cloudy_Daylight support="1"/>
<AWB_Twilight support="1"/>
<AWB_Shade support="1"/>
</AWB>
<Sence>
<Sence_Mode_Auto support="1"/>
<Sence_Mode_Action support="1"/>
<Sence_Mode_Portrait support="1"/>
<Sence_Mode_Landscape support="1"/>
<Sence_Mode_Night support="1"/>
<Sence_Mode_Night_Portrait support="1"/>
<Sence_Mode_Theatre support="1"/>
<Sence_Mode_Beach support="1"/>
<Sence_Mode_Snow support="1"/>
<Sence_Mode_Sunset support="1"/>
<Sence_Mode_Steayphoto support="1"/>
<Sence_Mode_Pireworks support="1"/>
<Sence_Mode_Sports support="1"/>
<Sence_Mode_Party support="1"/>
<Sence_Mode_Candlelight support="1"/>
<Sence_Mode_Barcode support="1"/>
<Sence_Mode_HDR support="1"/>
</Sence>
<Effect>
<Effect_None support="1"/>
<Effect_Mono support="1"/>
<Effect_Solarize support="1"/>
<Effect_Negative support="1"/>
<Effect_Sepia support="1"/>
<Effect_Posterize support="1"/>
<Effect_Whiteboard support="1"/>
<Effect_Blackboard support="1"/>
<Effect_Aqua support="1"/>
</Effect>
<FocusMode>
<Focus_Mode_Auto support="1"/>
<Focus_Mode_Infinity support="1"/>
<Focus_Mode_Marco support="1"/>
<Focus_Mode_Fixed support="1"/>
<Focus_Mode_Edof support="1"/>
<Focus_Mode_Continuous_Video support="0"/>
<Focus_Mode_Continuous_Picture support="1"/>
</FocusMode>
<FlashMode>
<Flash_Mode_Off support="1"/>
<Flash_Mode_On support="1"/>
<Flash_Mode_Torch support="1"/>
<Flash_Mode_Auto support="1"/>
<Flash_Mode_Red_Eye support="1"/>
</FlashMode>
<AntiBanding>
<Anti_Banding_Auto support="1"/>
<Anti_Banding_50HZ support="1"/>
<Anti_Banding_60HZ support="1"/>
<Anti_Banding_Off support="1"/>
</AntiBanding>
<HDR support="1"/>
<ZSL support="1"/>
<DigitalZoom support="1"/>
<Continue_SnapShot support="1"/>
<InterpolationRes resolution="0"/>
<PreviewSize width="1920" height="1080"/>
<FaceDetect support="0" MaxNum="1"/>
<DV>
<DV_QCIF name="qcif" width="176" height="144" fps="10" support="1"/>
<DV_QVGA name="qvga" width="320" height="240" fps="10" support="1"/>
<DV_CIF name="cif" width="352" height="288" fps="10" support="1"/>
<DV_VGA name="480p" width="640" height="480" fps="10" support="0"/>
<DV_480P name="480p" width="720" height="480" fps="10" support="0"/>
<DV_720P name="720p" width="1280" height="720" fps="10" support="1"/>
<DV_1080P name="1080p" width="1920" height="1080" fps="10" support="1"/>
</DV>
</SoftWareInfo>
</CamDevie>
</BoardFile>
```

主要修改的内容如下：

* Sensor 名称

 ```
<SensorName name="OV13850" ></SensorName>
 ```

该名字必须与 Sensor 驱动的名字一致,目前提供的 Sensor 驱动格式如下：

```
libisp_isi_drv_OV13850.so
```

*  Sensor 软件标识

```
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_1A"></SensorDevID>
```

注册标识不一致即可,可填写以下值：

```
CAMSYS_DEVID_SENSOR_1A
CAMSYS_DEVID_SENSOR_1B
CAMSYS_DEVID_SENSOR_2
```

*  采集控制器名称

```
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
```

目前只支持：

```
CAMSYS_DEVID_MARVIN
```

*  Sensor 所连接的主控 I2C 通道号

```
<SensorI2cBusNum busnum=3></SensorI2cBusNum>
```

具体通道号请参考摄像头原理图连接主控的 I2C 通道号。

*  Sensor 寄存器地址长度,单位：字节

```
<SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
```

*   Sensor 的 I2C 频率,单位：Hz，用于设置 I2C 的频率。

```
<SensorI2cRate rate="100000"></SensorI2cRate>
```

*   Sensor 输入时钟频率, 单位：Hz，用于设置摄像头的时钟。

```
<SensorMclk mclk="24000000"></SensorMclk>
```

*  Sensor AVDD 的 PMU LDO 名称。如果不是连接到 PMU，那么只需填写 NC。

```
<SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
```

*   Sensor DOVDD 的 PMU LDO 名称。

```
<SensorDovdd name="NC" min="18000000" max="18000000"></SensorDovdd>
```

如果不是连接到 PMU，那么只需填写 NC。注意 min 以及 max 值必须填写，这决定了 Sensor 的 IO 电压。

*  Sensor DVDD 的 PMU LDO 名称。

```
<SensorDvdd name="NC" min="0" max="0"></SensorDvdd>
```

如果不是连接到 PMU，那么只需填写 NC。

*  Sensor PowerDown 引脚。

```
<SensorGpioPwdn ioname="RK30_PIN2_PB6" active="0"></SensorGpioPwdn>
```

直接填写名称即可，active 填写休眠的有效电平。

*  Sensor Reset 引脚。

```
<SensorGpioRst ioname="RK30_PIN3_PB0" active="0"></SensorGpioRst>
```

直接填写名称即可，active 填写复位的有效电平。

*  Sensor Power 引脚。

```
<SensorGpioPwen ioname="RK30_PIN0_PB3" active="1"></SensorGpioPwen>
```

直接填写名称即可, active 填写电源有效电平。

*    选择 Sensor 作为前置还是后置。

```
<SensorFacing facing="front"></SensorFacing>
```

可填写 `front` 或 `back`。

*  Sensor 的接口方式

```
<SensorInterface mode=" MIPI "></SensorInterface>
```

可填写如下值：

```
CCIR601
CCIR656
MIPI
SMIA
```

*  Sensor 的镜像方式

```
<SensorMirrorFlip mirror="0"></SensorMirrorFlip>
```

目前暂不支持。

*  Sensor 的角度信息

```
<SensorOrientation orientation="0"></SensorOrientation>
```

*  物理接口设置

MIPI

```
<SensorPhy phyMode="CamSys_Phy_Mipi" lane="2" phyIndex="1" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
```

+ `phyMode`：Sensor 接口硬件连接方式，对 MIPI Sensor 来说，该值取 `CamSys_Phy_Mipi`
+ `lane`：Sensor mipi 接口数据通道数
+ `phyIndex`：Sensor MIPI 连接的主控 MIPI phy 编号
+ `sensorFmt`：Sensor 输出数据格式,目前仅支持 `CamSys_Fmt_Raw_10b`


编译内核需将 `drivers/media/video/rk_camsys` 驱动源码编进内核，其配置方法如下：

在内核源码目录下执行命令：

```
make menuconfig
```

然后将以下配置项打开：

```
Device Drivers  --->
 Multimedia support  --->
        camsys driver
         RockChip camera system driver  --->
                   camsys driver for marvin isp
                   camsys driver for cif
```

最后执行：

```
make ARCH=arm64 rk3399-firefly-aio.img
```

即可完成内核的编译。

## 调试方法

终端下可以直接修改 `/system/etc/cam_board.xml` 调试各参数并重启生效

## FAQs

1. 无法打开摄像头，首先确定 sensor I2C 是否通信。若不通则可检查 mclk 以及供电是否正常（Power/PowerDown/Reset/Mclk/I2cBus）分别排查
2. 支持列表ː
- 13Mː OV13850/IMX214-0AQH5
- 8Mː OV8825/OV8820/OV8858-Z(R1A)/OV8858-R2A
- 5Mː OV5648/OV5640
- 2Mː OV2680

详细资料可查询 SDK/RKDocs
