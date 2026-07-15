# Camera Use

## Introduction

Firefly-RK3128 development board has CIF interface, which supports front/back cameras.  
The following CIF cameras are supported in the driver directory kernel/drivers/media/video:  

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

This chapter mainly describes how to configure CIF.

## Configuration Steps

### Add Sensor Connection

Camera list is defined in file kernel/arch/arm/boot/dts/rk3128-cif-sensor.dtsi:
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

Here, we list the configuration of gc0329 back/front cameras with description:

* is_front: 0: back camera，1: front camera
* rockchip,powerdown: defines the camera's  powerdown gpio
* pwdn_active:  effect voltage level of powerdown
* mir: 0: don't support mirror, 1: support mirror
* resolution: maximum camera resolution
* orientation: 0: 0 degree， 90: 90 degrees， 180: 180 degrees， 270: 270 degrees
* i2c_add: Camera's I2C address
* i2c_rata: I2C frequency, with Hz unit.
* i2c_chl: I2C channel number
* cif_chl: cif controller info.rk312x only has cif0
* mclk_rate: sensor clock frequency, with MHz unit.

Some of the constants are defined in  kernel/arch/arm/mach-rockchip/rk_camera_sensor_info.h:
```
#define gc0329_FULL_RESOLUTION      0x30000            // 0.3 megapixel
#define gc0329_I2C_ADDR             0x62
#define gc0329_PWRDN_ACTIVE             0x01
#define gc0329_PWRSEQ                   sensor_PWRSEQ_DEFAULT
//Sensor power  active level define
#define PWR_ACTIVE_HIGH                  0x01
#define PWR_ACTIVE_LOW					 0x0
```

The resolutions are defined in function sensor_get_full_width_height in kernel/drivers/media/video/generic_sensor.h:
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

### Enable CIF Sensor Device

To enable CIF, add the following lines to  kernel/arch/arm/boot/dts/rk3128-fireprime.dts:
```
&rk3128_cif_sensor {
	status = "okay";
};
```