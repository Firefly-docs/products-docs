# MIPI CSI 使用

## 板载资源

AIO-3288C 开发板有 MIPI 摄像头接口，摄像头图像处理能力达到 4416x3312 像素，支持 4K 视频录制。此外，开发板还支持 USB 摄像头。

本文以 OV13850 摄像头为例，讲解 MIPI 摄像头在该开发板上的配置过程。

## 相关代码目录

与摄像头相关的代码目录如下：

```c
Android：
`- hardware/rockchip/camera
   |- Config
   |  `- cam_board.xml          // 摄像头的参数设置
   |- CameraHal                 // 摄像头的 HAL 源码
   `- SiliconImage              // ISP 库，包括所有支持模组的驱动源码
      `- isi/drv/OV13850        // OV13850 模组的驱动源码
         `- calib/OV13850.xml   // OV13850 模组的调校参数
```

```c
Kernel：
|- kernel/drivers/media/video/rk_camsys  // CamSys 驱动源码
`- kernel/include/media/camsys_head.h
```

!INCLUDE "include/AIO-3288C/driver_mipi_csi_interface.mdpp"

## 配置步骤

### 配置 Android

修改 `hardware/rockchip/camera/Config/cam_board_rk3288_aio-3288c.xml` 来注册摄像头：

```xml
<?xml version="1.0" ?>
    <BoardFile>
     <BoardXmlVersion version="v0.7.0">
        </BoardXmlVersion>
          <CamDevie>
            <HardWareInfo>
                <Sensor>
                    <SensorName name="OV13850" ></SensorName>
                    <SensorDevID IDname="CAMSYS_DEVID_SENSOR_1B"></SensorDevID>
                    <SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
                    <SensorI2cBusNum busnum="3"></SensorI2cBusNum>
                    <SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
                    <SensorI2cRate rate="100000"></SensorI2cRate>
                    <SensorMclk mclk="24000000"></SensorMclk>
                    <SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
                    <SensorDovdd name="NC" min="18000000" max="18000000"></SensorDovdd>
                    <SensorDvdd name="NC" min="0" max="0"></SensorDvdd>
                    <SensorGpioPwdn ioname="RK30_PIN2_PB6" active="0"></SensorGpioPwdn>
                    <SensorGpioRst ioname="RK30_PIN3_PB0" active="0"></SensorGpioRst>
                    <SensorGpioPwen ioname="RK30_PIN0_PB3" active="1"></SensorGpioPwen>
                    <SensorFacing facing="front"></SensorFacing>
                    <SensorInterface interface="MIPI"></SensorInterface>
                    <SensorMirrorFlip mirror="0"></SensorMirrorFlip>
                    <SensorOrientation orientation="0"></SensorOrientation>
                    <SensorPowerupSequence seq="1234"></SensorPowerupSequence>
                    <SensorFovParemeter h="60.0" v="60.0"></SensorFovParemeter>
                    <SensorAWB_Frame_Skip fps="15"></SensorAWB_Frame_Skip>
                    <SensorPhy phyMode="CamSys_Phy_Mipi" lane="2"  phyIndex="1" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
                </Sensor>
                <VCM>
                    <VCMDrvName name="BuiltInSensor"></VCMDrvName>
                    <VCMName name="NC"></VCMName>
                    <VCMI2cBusNum busnum="3"></VCMI2cBusNum>
                    <VCMI2cAddrByte byte="0"></VCMI2cAddrByte>
                    <VCMI2cRate rate="0"></VCMI2cRate>
                    <VCMVdd name="NC" min="0" max="0"></VCMVdd>
                    <VCMGpioPwdn ioname="NC" active="0"></VCMGpioPwdn>
                    <VCMGpioPower ioname="NC" active="0"></VCMGpioPower>
                    <VCMCurrent start="20" rated="80" vcmmax="100" stepmode="13"  drivermax="100"></VCMCurrent>
                </VCM> 
                <Flash>
                     <FlashName name="Internal"></FlashName>
                     <FlashI2cBusNum busnum="0"></FlashI2cBusNum>
                     <FlashI2cAddrByte byte="0"></FlashI2cAddrByte>
                     <FlashI2cRate rate="0"></FlashI2cRate>
                     <FlashTrigger ioname="NC" active="0"></FlashTrigger>
                     <FlashEn ioname="NC" active="0"></FlashEn>
                     <FlashModeType mode="1"></FlashModeType>
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
                <Focus_Mode_Auto support="1"></Focus_Mode_Auto>
                <Focus_Mode_Infinity support="1"></Focus_Mode_Infinity>
                <Focus_Mode_Marco support="1"></Focus_Mode_Marco>
                <Focus_Mode_Fixed support="1"></Focus_Mode_Fixed>
                <Focus_Mode_Edof support="1"></Focus_Mode_Edof>
                <Focus_Mode_Continuous_Video support="0"></Focus_Mode_Continuous_Video>
                <Focus_Mode_Continuous_Picture support="1"></Focus_Mode_Continuous_Picture>
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
                <DV_VGA name="480p" width="640" height="480" fps="10" support="0"></DV_VGA>
                <DV_480P name="480p" width="720" height="480" fps="10" support="0"></DV_480P>
                <DV_720P name="720p" width="1280" height="720" fps="10" support="1"></DV_720P>
                <DV_1080P name="1080p" width="1920" height="1080" fps="10" support="1"></DV_1080P>
            </DV>
        </SoftWareInfo>
    </CamDevie>
</BoardFile>
```

主要修改的内容如下：

* Sensor 名称

```xml
<SensorName name="OV13850" ></SensorName>
 ```

该名字必须与 Sensor 驱动的名字一致,目前提供的 Sensor 驱动格式如下：

```c
libisp_isi_drv_OV13850.so
```

用户可在编译 Android 完成后在目录 `out/target/product/rk3288_aio_3288j_box/system/lib/hw/` 下找到该摄像头驱动文件。

* Sensor 软件标识

```xml
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_1A"></SensorDevID>
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
<SensorGpioPwdn ioname="RK30_PIN2_PB6" active="0"></SensorGpioPwdn>
```

直接填写名称即可，active 填写休眠的有效电平。

* Sensor Reset 引脚。


```xml
<SensorGpioRst ioname="RK30_PIN3_PB0" active="0"></SensorGpioRst>
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

可填写 "front" 或 "back"。

* Sensor 的接口方式

```xml
<SensorInterface mode="MIPI"></SensorInterface>
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
    + MIPI

```xml
<SensorPhy phyMode="CamSys_Phy_Mipi" lane="2" phyIndex="1" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
```

1. phyMode：Sensor 接口硬件连接方式，对 MIPI Sensor 来说，该值取 "CamSys_Phy_Mipi"
2. lane：Sensor mipi 接口数据通道数
3. phyIndex：Sensor mipi 连接的主控 mipi phy 编号
4. sensorFmt：Sensor 输出数据格式,目前仅支持 CamSys_Fmt_Raw_10b

### 配置内核

在配置原理中提到，GPIO7_B4 需要在 DTS 和驱动中配置。其配置方法如下：

(1). DTS 文件添加 GPIO7_B4 配置属性

在 `kernel/arch/arm/boot/dts/rk3288.dtsi` 文件中添加 gpios-cifpower 属性，如下所示：

```dts
isp： isp@ff910000{
    compatible = "firefly,isp";
    ...
    gpios-cifpower = ;
    ...
    status = "okay";
};
```

(2). 驱动中配置 CIF_POWER

在 `kernel/drivers/media/video/rk_camsys/camsys_drv.c` 中读取 gpios-cifpower ，并设置该引脚，使能 CIF_POWER，在 probe 函数 camsys_platform_probe() 中添加如下所示：

```c
enum of_gpio_flags flags;
int cifpower_io;
int io_ret;
cifpower_io = of_get_named_gpio_flags(dev->of_node, "gpios-cifpower", 0, &flags);
camsys_trace(1, "1-gpios-cifpower： gpio=%d", cifpower_io);
if(gpio_is_valid(cifpower_io)){
     cifpower_io = of_get_named_gpio_flags(dev->of_node, "gpios-cifpower", 0, &flags);
     camsys_trace(1, "gpios-cifpower： gpio_request");
     io_ret = gpio_request(cifpower_io,"cifpower");
     camsys_trace(1, "1-gpios-cifpower： gpio_request=%d", io_ret);
     if(io_ret < 0){
       camsys_err("Request %s(%d) failed","cifpower", cifpower_io);
     }
     else{
        gpio_direction_output(cifpower_io, 1);
        gpio_set_value(cifpower_io, 1);
        camsys_trace(1, "gpios-cifpower： %d high", cifpower_io);
    }
}
```

(3). 编译内核

需将 `drivers/media/video/rk_camsys` 驱动源码编进内核，其配置方法如下：

在内核源码目录下执行命令：

```bash
make menuconfig
```

然后将以下配置项打开：

```c
Device Drivers  --->
Multimedia support  --->
        camsys driver
         RockChip camera system driver  --->
                   camsys driver for marvin isp
                   camsys driver for cif
```

最后执行：

```bash
make firefly-rk3288-aio-3288c.img
```

即可完成内核的编译。