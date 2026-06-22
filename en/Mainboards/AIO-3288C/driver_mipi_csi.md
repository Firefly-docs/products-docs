# MIPI CSI Use

## Onboard resources

The AIO-3288C development board has the MIPI camera interface. The camera’s image processing capability reaches 4416x3312 pixels and supports 4K video recording. In addition, the development board also supports a USB camera.

This article takes the OV13850 camera as an example to explain the configuration process of the MIPI camera on the development board.

## Related code catalog

The code catalog related to camera is as follows:

```c
Android:
`- hardware/rockchip/camera
   |- Config
   |  `- cam_board.xml          // parameter setting of camera
   |- CameraHal                 // HAL source code of camera
   `- SiliconImage              // ISP library, including all drivers source codes supporting module
      `- isi/drv/OV13850        // Driver source code of OV13850 module
         `- calib/OV13850.xml   // Adjustment parameters of OV13850 module
```

```c
Kernel:
|- kernel/drivers/media/video/rk_camsys  // CamSys driver source code
`- kernel/include/media/camsys_head.h
```

!INCLUDE "include/AIO-3288C/driver_mipi_csi_interface.mdpp"

## Configuration steps

### Configure Android

Modify `hardware/rockchip/camera/Config/cam_board_rk3288_aio-3288c.xml` to register camera:

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

The main modified contents are as follows:

* Sensor name

```xml
<SensorName name="OV13850" ></SensorName>
 ```

This name must coincide to the name of Sensor driver, and the format of Sensor driver currently provided is as follows:

```c
libisp_isi_drv_OV13850.so
```

The user can find the driver file of this camera under the catalog `out/target/product/rk3288_aio_3288j_box/system/lib/hw/` after completing Android compilation.

* Sensor software identification

```xml
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_1A"></SensorDevID>
```

Any inconsistent registration identification is okay, the following value can be filled in:

```xml
CAMSYS_DEVID_SENSOR_1A
CAMSYS_DEVID_SENSOR_1B
CAMSYS_DEVID_SENSOR_2
```

* Name of acquisition controller

```xml
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
```

At present, it only supports:

```xml
CAMSYS_DEVID_MARVIN
```

* Number of I2C channel connected to master control of Sensor

```xml
<SensorI2cBusNum busnum="3"></SensorI2cBusNum>  
```

For the specific channel number, please refer to the number of I2C channel connected to master control on the schematic diagram of camera.

* Sensor register address size, unit: bytes

```xml
<SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
```

* I2C frequency of Sensor, unit: Hz, which is used for setting I2C frequency.

```xml
<SensorI2cRate rate="100000"></SensorI2cRate>
```

* Sensor input clock frequency, unit: Hz, which is used for setting camera clock.

```xml
<SensorMclk mclk="24000000"></SensorMclk>
```

* PMU LDO name of Sensor AVDD. If it cannot be connected to PMU, fill in NC only.

```xml
<SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
```

* PMU LDO name of Sensor DOVDD.

```xml
<SensorDovdd name="NC" min="18000000" max="18000000"></SensorDovdd>
```

If it is not connected to PMU, fill in NC only. The values of min and max must be filled in, which decide the IO voltages of Sensor.

* PMU LDO name of Sensor DVDD.

```xml
<SensorDvdd name="NC" min="0" max="0"></SensorDvdd>
```

If it is not connected to PMU, fill in NC only.

* Sensor PowerDown pin.

```xml
<SensorGpioPwdn ioname="RK30_PIN2_PB6" active="0"></SensorGpioPwdn>
```

Directly fill in the name, and fill in the active level of sleep in active field.

* Sensor Reset pin.


```xml
<SensorGpioRst ioname="RK30_PIN3_PB0" active="0"></SensorGpioRst>
```


Directly fill in the name, and fill in the active level of reset in active field.

* Sensor Power pin.

```xml
<SensorGpioPwen ioname="RK30_PIN0_PB3" active="1"></SensorGpioPwen>
```

Directly fill in the name, and fill in the active level of power in active field.

* Select front or back for the Sensor.

```xml
<SensorFacing facing="front"></SensorFacing>
```

"Front" or "back" can be filled in.

* Sensor interface mode

```xml
<SensorInterface mode="MIPI"></SensorInterface>
```

The following value can be filled in:

```xml
CCIR601
CCIR656
MIPI
SMIA
```

* Sensor image mode

```xml
<SensorMirrorFlip mirror="0"></SensorMirrorFlip>
```

Do not support at present.

* Sensor angle information

```xml
<SensorOrientation orientation="0"></SensorOrientation>
```

* Physical interface settings
    + MIPI

```xml
<SensorPhy phyMode="CamSys_Phy_Mipi" lane="2" phyIndex="1" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
```

1. phyMode: Sensor port hardware connection mode, for MIPI Sensor, this value shall take "CamSys_Phy_Mipi"
2. lane: number of data channels in Sensor mipi interface
3. phyIndex: mipi phy number of master control connected to Sensor mipi
4. sensorFmt: Sensor output data format, which only supports CamSys_Fmt_Raw_10b at present

### Configuration kernel

The configuration principles mention that GPIO7_B4 should be configured in DTS and driver. The configuration methods are as follows:

(1). Add GPIO7_B4 configuration properties in DTS file

Add gpios-cifpower property in `kernel/arch/arm/boot/dts/rk3288.dtsi` file, as shown below:

```dts
isp:  isp@ff910000{
    compatible = "firefly,isp";
    ...
    gpios-cifpower = ;
    ...
    status = "okay";
};
```

(2). Configure CIF_POWER in driver

Read gpios-cifpower in `kernel/drivers/media/video/rk_camsys/camsys_drv.c`, set this pin, enable CIF_POWER, and add in probe function camsys_platform_probe(), as shown below:

```c
enum of_gpio_flags flags;
int cifpower_io;
int io_ret;
cifpower_io = of_get_named_gpio_flags(dev->of_node, "gpios-cifpower", 0, &flags);
camsys_trace(1, "1-gpios-cifpower:  gpio=%d", cifpower_io);
if(gpio_is_valid(cifpower_io)){
     cifpower_io = of_get_named_gpio_flags(dev->of_node, "gpios-cifpower", 0, &flags);
     camsys_trace(1, "gpios-cifpower:  gpio_request");
     io_ret = gpio_request(cifpower_io,"cifpower");
     camsys_trace(1, "1-gpios-cifpower:  gpio_request=%d", io_ret);
     if(io_ret < 0){
       camsys_err("Request %s(%d) failed","cifpower", cifpower_io);
     }
     else{
        gpio_direction_output(cifpower_io, 1);
        gpio_set_value(cifpower_io, 1);
        camsys_trace(1, "gpios-cifpower:  %d high", cifpower_io);
    }
}
```

(3). Compile kernel

Compile the driver source code `drivers/media/video/rk_camsys` into the kernel; its configuration methods are as follows:

Execute command under kernel source code catalog:

```bash
make menuconfig
```

Then, open the following configuration item:

```c
Device Drivers  --->
Multimedia support  --->
        camsys driver
         RockChip camera system driver  --->
                   camsys driver for marvin isp
                   camsys driver for cif
```

Finally, execute:

```bash
make firefly-rk3288-aio-3288c.img
```

The compilation of kernel is completed.