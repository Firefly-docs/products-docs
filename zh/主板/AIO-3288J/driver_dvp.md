# DVP 使用

## 板载资源

AIO-3288J-Reload 开发板带有一个 DVP 摄像头接口。本文以 OV5640 摄像头为例，讲解在该开发板上的配置过程。

## 相关代码目录

与摄像头相关的代码目录如下：

* Android：

```bash
`- hardware/rockchip/camera
    |- Config
    |  `- cam_board_rk3288.xml      # 摄像头的参数设置
    |- CameraHal                    # 摄像头的 HAL 源码
    `- SiliconImage                 # ISP 库，包括所有支持模组的驱动源码
       `- isi/drv/OV5640            # OV5640 模组的驱动源码
            `- calib/OV5640.xml     # OV5640 模组的调校参数
```

* kernel

```bash
|- kernel/drivers/media/video/rk_camsys  # CamSys 驱动源码
 `- kernel/include/media/camsys_head.h
```

## 配置原理

设置摄像头相关的引脚和时钟，即可完成配置过程。 从以下摄像头接口原理图可知，需要配置的引脚有：VCC28_DVP、VCC18_DVP、VCCIO_YUV、PWDN(FLASH0_CLE)、RESET 和 XCLK1。

![](../../../rk3288_img/dvp_interface.png)
![](../../../rk3288_img/dvp_power_1.png)
![](../../../rk3288_img/dvp_power_2.png)
![](../../../rk3288_img/dvp_power_3.png)

PWDN、RST 对应 GPIO3_B4、GPIO2_B7 引脚

## 配置步骤

### 配置 Android

修改 hardware/rockchip/camera/Config/cam_board_rk3288.xml 来注册摄像头：

```xml
<?xml version="1.0" ?>
<BoardFile>
        <BoardXmlVersion version="v0.0xd.0"></BoardXmlVersion>
    <CamDevie>
        <HardWareInfo>
            <Sensor>
                <SensorName name="OV5640" ></SensorName>
                <SensorDevID IDname="CAMSYS_DEVID_SENSOR_2"></SensorDevID>
                <SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
                <SensorI2cBusNum busnum="3"></SensorI2cBusNum>
                <SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
                <SensorI2cRate rate="100000"></SensorI2cRate>
                <SensorMclk mclk="24000000"></SensorMclk>
                <SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
                <SensorDovdd name="NC" min="18000000" max="18000000"></SensorDovdd>
                <SensorDvdd name="NC" min="0" max="0"></SensorDvdd>
                <SensorGpioPwdn ioname="RK30_PIN3_PB4" active="1"></SensorGpioPwdn>
                <SensorGpioRst ioname="RK30_PIN2_PB7" active="0"></SensorGpioRst>
                <SensorGpioPwen ioname="RK30_PIN0_PB3" active="1"></SensorGpioPwen>
                <SensorFacing facing="front"></SensorFacing>
                <SensorInterface mode="CCIR601"></SensorInterface>
                <SensorMirrorFlip mirror="0"></SensorMirrorFlip>
                <SensorOrientation orientation="0"></SensorOrientation>
                <SensorPowerupSequence seq="1234">
                <SensorFovParemeter h="60.0" v="60.0"></SensorFovParemeter>
                <SensorAWB_Frame_Skip fps="15"></SensorAWB_Frame_Skip>
                <SensorPhy phyMode="CamSys_Phy_Cif" sensor_d0_to_cif_d="0" cif_num="0" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
             </Sensor>
             <VCM>
                <VCMDrvName name="NC"></VCMDrvName>
                <VCMName name="NC"></VCMName>
                <VCMI2cBusNum busnum="0"></VCMI2cBusNum>
                <VCMI2cAddrByte byte="0"></VCMI2cAddrByte>
                <VCMI2cRate rate="0"></VCMI2cRate>
                <VCMVdd name="NC" min="0" max="0"></VCMVdd>
                <VCMGpioPwdn ioname="NC" active="0"></VCMGpioPwdn>
                <VCMGpioPower ioname="NC" active="0"></VCMGpioPower>
<VCMCurrent start="20" rated="80" vcmmax="100" stepmode="13"  drivermax="100"></VCMCurrent>
            </VCM>
            <Flash>
                <FlashName name="NC"></FlashName>
                <FlashI2cBusNum busnum="0"></FlashI2cBusNum>
                <FlashI2cAddrByte byte="0"></FlashI2cAddrByte>
                <FlashI2cRate rate="0"></FlashI2cRate>
                <FlashTrigger ioname="NC" active="1"></FlashTrigger>
                <FlashEn ioname="NC" active="1"></FlashEn>
                <FlashModeType mode="0"></FlashModeType>
                <FlashLuminance luminance="0"></FlashLuminance>
                <FlashColorTemp colortemp="0"></FlashColorTemp>
            </Flash>
    </HardWareInfo>
        <SoftWareInfo>
            <AWB>
                <AWB_Auto support="1"></AWB_Auto>
                <AWB_Incandescent support="1"></AWB_Incandescent>
                <AWB_Fluorescent support="1"></AWB_Fluorescent>
                <AWB_Warm_Fluorescent support="1"></AWB_Warm_Fluorescent>
                <AWB_Daylight support="1"></AWB_Daylight>
                <AWB_Cloudy_Daylight support="1"></AWB_Cloudy_Daylight>
                <AWB_Twilight support="1"></AWB_Twilight>
                <AWB_Shade support="1"></AWB_Shade>
            </AWB>
            <Sence>
                <Sence_Mode_Auto support="1"></Sence_Mode_Auto>
                <Sence_Mode_Action support="1"></Sence_Mode_Action>
                <Sence_Mode_Portrait support="1"></Sence_Mode_Portrait>
                <Sence_Mode_Landscape support="1"></Sence_Mode_Landscape>
                <Sence_Mode_Night support="1"></Sence_Mode_Night>
                <Sence_Mode_Night_Portrait support="1"></Sence_Mode_Night_Portrait>
                <Sence_Mode_Theatre support="1"></Sence_Mode_Theatre>
                <Sence_Mode_Beach support="1"></Sence_Mode_Beach>
                <Sence_Mode_Snow support="1"></Sence_Mode_Snow>
                <Sence_Mode_Sunset support="1"></Sence_Mode_Sunset>
                <Sence_Mode_Steayphoto support="1"></Sence_Mode_Steayphoto>
                <Sence_Mode_Pireworks support="1"></Sence_Mode_Pireworks>
                <Sence_Mode_Sports support="1"></Sence_Mode_Sports>
                <Sence_Mode_Party support="1"></Sence_Mode_Party>
                <Sence_Mode_Candlelight support="1"></Sence_Mode_Candlelight>
                <Sence_Mode_Barcode support="1"></Sence_Mode_Barcode>
                <Sence_Mode_HDR support="1"></Sence_Mode_HDR>
            </Sence>
            <Effect>
                <Effect_None support="1"></Effect_None>
                <Effect_Mono support="1"></Effect_Mono>
                <Effect_Solarize support="1"></Effect_Solarize>
                <Effect_Negative support="1"></Effect_Negative>
                <Effect_Sepia support="1"></Effect_Sepia>
                <Effect_Posterize support="1"></Effect_Posterize>
                <Effect_Whiteboard support="1"></Effect_Whiteboard>
                <Effect_Blackboard support="1"></Effect_Blackboard>
                <Effect_Aqua support="1"></Effect_Aqua>
            </Effect>
            <FocusMode>
                <Focus_Mode_Auto support="0"></Focus_Mode_Auto>
                <Focus_Mode_Infinity support="0"></Focus_Mode_Infinity>
                <Focus_Mode_Marco support="0"></Focus_Mode_Marco>
                <Focus_Mode_Fixed support="0"></Focus_Mode_Fixed>
                <Focus_Mode_Edof support="0"></Focus_Mode_Edof>
                <Focus_Mode_Continuous_Video support="0"></Focus_Mode_Continuous_Video>
                <Focus_Mode_Continuous_Picture support="0"></Focus_Mode_Continuous_Picture>
             </FocusMode>
            <FlashMode>
                <Flash_Mode_Off support="1"></Flash_Mode_Off>
                <Flash_Mode_On support="1"></Flash_Mode_On>
                <Flash_Mode_Torch support="1"></Flash_Mode_Torch>
                <Flash_Mode_Auto support="1"></Flash_Mode_Auto>
                <Flash_Mode_Red_Eye support="1"></Flash_Mode_Red_Eye>
            </FlashMode>
            <AntiBanding>
                <Anti_Banding_Auto support="1"></Anti_Banding_Auto>
                <Anti_Banding_50HZ support="1"></Anti_Banding_50HZ>
                <Anti_Banding_60HZ support="1"></Anti_Banding_60HZ>
                <Anti_Banding_Off support="1"></Anti_Banding_Off>
            </AntiBanding>
            <HDR support="1"></HDR>
            <ZSL support="1"></ZSL>
            <DigitalZoom support="1"></DigitalZoom>
            <Continue_SnapShot support="1"></Continue_SnapShot>
            <PreviewSize width="800" height="600"></PreviewSize>
            <DV>
                <DV_QCIF name="qcif" width="176" height="144" fps="10" support="1"></DV_QCIF>
                <DV_QVGA name="qvga" width="320" height="240" fps="10" support="1"></DV_QVGA>
                <DV_CIF name="cif" width="352" height="288" fps="10" support="1"></DV_CIF>
                <DV_VGA name="480p" width="640" height="480" fps="10" support="1"></DV_VGA>
                <DV_480P name="480p" width="720" height="480" fps="10" support="0"></DV_480P>
                <DV_720P name="720p" width="1280" height="720" fps="10" support="0"></DV_720P>
                <DV_1080P name="1080p" width="1920" height="1080" fps="10" support="0"></DV_1080P>
            </DV>
        </SoftWareInfo>
    </CamDevie>
</BoardFile>
```

主要修改的内容如下：

* Sensor 名称

```xml
<SensorName name="OV5640" ></SensorName>
```

该名字必须与 Sensor 驱动的名字一致,目前提供的 Sensor 驱动格式如下： libisp_isi_drv_OV5640.so
用户可在编译 Android 完成后在目录 out/target/product/rk3288_box/system/lib/hw/ 下找到该摄像头驱动文件。

* Sensor 软件标识

```xml
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_2"></SensorDevID>
```

注册标识不一致即可,可填写以下值：

```xml
CAMSYS_DEVID_SENSOR_1A
CAMSYS_DEVID_SENSOR_1B
CAMSYS_DEVID_SENSOR_2
```

* 采集控制器名称

```xml
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
```

目前只支持：

```xml
CAMSYS_DEVID_MARVIN
```

* Sensor 所连接的主控 I2C 通道号

```xml
<SensorI2cBusNum busnum="3"></SensorI2cBusNum>
```

具体通道号请参考摄像头原理图连接主控的 I2C 通道号。

* Sensor 寄存器地址长度,单位：字节

```xml
<SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
```

* Sensor 的 I2C 频率,单位：Hz，用于设置 I2C 的频率。

```xml
<SensorI2cRate rate="100000"></SensorI2cRate>
```

* Sensor 输入时钟频率, 单位：Hz，用于设置摄像头的时钟。

```xml
<SensorMclk mclk="24000000"></SensorMclk>
```

* Sensor AVDD 的 PMU LDO 名称。如果不是连接到 PMU，那么只需填写 NC。

```xml
<SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
```

* Sensor DOVDD 的 PMU LDO 名称。

```xml
<SensorDovdd name="NC" min="18000000" max="18000000"></SensorDovdd>
```

如果不是连接到 PMU，那么只需填写 NC。注意 min 以及 max 值必须填写，这决定了 Sensor 的 IO 电压。

* Sensor DVDD 的 PMU LDO 名称。

```xml
<SensorDvdd name="NC" min="0" max="0"></SensorDvdd>
```

如果不是连接到 PMU，那么只需填写 NC。

* Sensor PowerDown 引脚。

```xml
<SensorGpioPwdn ioname="RK30_PIN3_PB4" active="1"></SensorGpioPwdn>
```

直接填写名称即可，active 填写休眠的有效电平。

* Sensor Reset 引脚

```xml
<SensorGpioRst ioname="RK30_PIN2_PB7" active="0"></SensorGpioRst>
```

直接填写名称即可，active 填写复位的有效电平。

* Sensor Power 引脚。

```xml
<SensorGpioPwen ioname="RK30_PIN0_PB3" active="1"></SensorGpioPwen>
```

直接填写名称即可, active 填写电源有效电平。

* 选择 Sensor 作为前置还是后置。

```xml
<SensorFacing facing="front"></SensorFacing>
```

可填写 "front" 或 "back"

* Sensor 的接口方式

```xml
<SensorInterface mode="CCIR601"></SensorInterface>
```

可填写如下值：

```xml
CCIR601
CCIR656
MIPI
SMIA
```

* Sensor 的镜像方式

```xml
<SensorMirrorFlip mirror="0"></SensorMirrorFlip>
```

目前暂不支持。

* Sensor 的角度信息

```xml
<SensorOrientation orientation="0"></SensorOrientation>
```

* 物理接口设置

```xml
<SensorPhy phyMode="CamSys_Phy_Cif" sensor_d0_to_cif_d ="0" cif_num="0" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
```

1. phyMode： Sensor 接口硬件连接方式，DVP Sensor 接口则为：CamSys_Phy_Cif
2. sensor_d0_to_cif_d：Sensor DVP 输出数据位 D0 对应连接的主控 DVP 接口的数据位号码  
3. cif_num：Sensor DVP 连接到主控 DVP 接口编号  
4. sensorFmt：Sensor 输出的数据格式,目前版本仅支持填写 CamSys_Fmt_Raw_10b

### 编译

在内核源码目录下执行命令：

```bash
make menuconfig
```

然后将以下配置项打开：

```bash
Device Drivers  --->

<*>Multimedia support  --->
       <*>camsys driver
        RockChip camera system driver  --->
                 <*> camsys driver for marvin isp
                 < > camsys driver for cif
```

最后执行：

```bash
make firefly-rk3288-reload.img  
```

即可完成内核的编译。