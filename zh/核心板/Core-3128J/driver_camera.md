# Camera 使用

## 前言

Firefly-RK3128 开发板上有 CIF 接口，支持 CIF 摄像头。
在 kernel/drivers/media/video 目录里有以下 CIF 摄像头的驱动：

* gc0307
* gc0308
* gc0309
* gc0328
* gc0329
* gc2015
* gc2035
* gt2005
* hm2057
* hm5065
* mt9p111
* nt99160
* nt99240
* ov2659
* ov5640
* sp0838
* sp2518


本章主要描述如何配置 CIF。

## 配置步骤

### 加入传感器的连接信息

在 kernel/arch/arm/boot/dts/rk3128-cif-sensor.dtsi 定义摄像头列表：

```
rk3128_cif_sensor: rk3128_cif_sensor{
        compatible = "rockchip,sensor";
        status = "disabled";
        CONFIG_SENSOR_POWER_IOCTL_USR 		= <1>;
        CONFIG_SENSOR_RESET_IOCTL_USR 		= <0>;
        CONFIG_SENSOR_POWERDOWN_IOCTL_USR	= <0>;
        CONFIG_SENSOR_FLASH_IOCTL_USR	  	= <0>;
        CONFIG_SENSOR_AF_IOCTL_USR	   		= <0>;
        // ... skip some modules
        gc0329{
            is_front = <1>;
            rockchip,powerdown = <&gpio3 GPIO_D7 GPIO_ACTIVE_HIGH>;
            pwdn_active = <gc0329_PWRDN_ACTIVE>;
            #rockchip,power = <>;
            pwr_active = <PWR_ACTIVE_HIGH>;
            #rockchip,reset = <>;
            #rst_active = <>;
            #rockchip,flash = <>;
            #rockchip,af = <>;
            mir = <0>;
            flash_attach = <0>;
            resolution = <gc0329_FULL_RESOLUTION>;
            powerup_sequence = <gc0329_PWRSEQ>;
            orientation = <0>;
            i2c_add = <gc0329_I2C_ADDR>;
            i2c_rata = <100000>;
            i2c_chl = <0>;
            cif_chl = <0>;
            mclk_rate = <24>;
		};
        gc0329_b {
            is_front = <0>;
            rockchip,powerdown = <&gpio3 GPIO_B3 GPIO_ACTIVE_HIGH>;
            pwdn_active = <gc0329_PWRDN_ACTIVE>;
            #rockchip,power = <>;
            pwr_active = <PWR_ACTIVE_HIGH>;
            #rockchip,reset = <>;
            #rst_active = <>;
            #rockchip,flash = <>;
            #rockchip,af = <>;
            mir = <0>;
            flash_attach = <0>;
            resolution = <gc0329_FULL_RESOLUTION>;
            powerup_sequence = <gc0329_PWRSEQ>;
            orientation = <0>;
            i2c_add = <gc0329_I2C_ADDR>;
            i2c_rata = <100000>;
            i2c_chl = <0>; // <0>;
            cif_chl = <0>;
            mclk_rate = <24>;
        };
};
```

这里列出 gc0329 双摄像头模组的配置，其属性解释如下：

* is_front: 0: 后置摄像头，1: 前置摄像头
* rockchip,powerdown: 定义该摄像头对应的 powerdown gpio
* pwdn_active: powerdown 的有效电平
* mir: 0: 不支持镜像，1: 支持镜像，
* resolution: 最大的分辨率
* orientation: 旋转角度，0: 0 度， 90: 90 度， 180: 180 度， 270: 270 度
* i2c_add: 摄像头的 I2C 地址
* i2c_rata: I2C 频率，单位 Hz
* i2c_chl: I2C 通道号
* cif_chl: cif 控制器信息，rk312x 仅有 cif0
* mclk_rate: 传感器时钟频率，单位：MHz

其中的常量值都在 kernel/arch/arm/mach-rockchip/rk_camera_sensor_info.h 中定义：

```
#define gc0329_FULL_RESOLUTION      0x30000            // 0.3 megapixel
#define gc0329_I2C_ADDR             0x62
#define gc0329_PWRDN_ACTIVE             0x01
#define gc0329_PWRSEQ                   sensor_PWRSEQ_DEFAULT
//Sensor power  active level define
#define PWR_ACTIVE_HIGH                  0x01
#define PWR_ACTIVE_LOW					 0x0
```

分辨率对应的图像长宽，在 kernel/drivers/media/video/generic_sensor.h 的 sensor_get_full_width_height 函数里定义：

```
static inline int sensor_get_full_width_height(int full_resolution, unsigned short *w, unsigned short *h)
{
    switch (full_resolution)
    {
        case 0x30000:
        {
            *w = 640;
            *h = 480;
            break;
        }

        case 0x100000:
        {
            *w = 1024;
            *h = 768;
            break;
        }
        //...
    }
}
```

### 启用 CIF 传感器设备

要启用 CIF ，只需在 kernel/arch/arm/boot/dts/rk3128-fireprime.dts 加入：
```
&rk3128_cif_sensor{
    status = "okay";
};
```