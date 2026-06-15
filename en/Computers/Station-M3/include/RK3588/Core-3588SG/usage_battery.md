# BATTERY 

## Introduction
The AIO-3588SG has a battery port, as shown in the following figure：

![](../../../rk3588_img/Station-M3/usage_battery_interface.png) 


The battery should meet the following conditions:
* Combination method: 2S1P
* Nominal voltage: 7.4V
* Maximum discharge voltage: 8.4V
* Minimum discharge voltage: 6V

## Fuel Gauge Software Configuration

The fuel gauge chip is a device dedicated to battery monitoring, which can detect the remaining power, voltage, and battery temperature of the battery. Generally, a fuel gauge matches a battery. If a battery is replaced, the fuel gauge needs to be reconfigured. <br>
The fuel gauge cw2017 used by AIO-3588SG is located on the development board, dts reference `./kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dtsi`

```
    cw2017: cw2017@63 {
        status = "okay";
        compatible = "cellwise,cw2017";
        reg = <0x63>;

        //Battery parameters, you need to send the battery to the fuel gauge manufacturer, and scan the battery parameters
        cellwise,battery-profile = /bits/ 8
            <
			0x3C 0x00 0x00 0x00 0x00 0x00 0x00 0x00
			0xB0 0xC4 0xBF 0xB9 0x9B 0x97 0xE0 0xCF
			0xC1 0xCD 0xBB 0x9D 0x88 0x7C 0x65 0x56
			0x52 0x50 0x4E 0x97 0x79 0xD2 0xDE 0xFF
			0xE5 0xB4 0x71 0x7C 0xB0 0xC5 0xAE 0x93
			0x9D 0xB5 0xCF 0xD5 0xC6 0xB0 0x99 0x89
			0x82 0x85 0x91 0xA8 0xC1 0xC9 0xB0 0x43
			0x00 0x00 0x90 0x02 0x00 0x00 0x00 0x00
			0x00 0x00 0x64 0x00 0x00 0x00 0x00 0x00
			0x00 0x00 0x00 0x00 0x00 0x00 0x00 0xA2
            >;
        cellwise,dual-cell;
        cellwise,monitor-interval-ms = <5000>;

        // Set low battery reminder, when the battery power reaches 10%, it will send an interrupt
        cellwise,alert-level = <10>;
        // Battery capacity, 5000mah
        cellwise,design-capacity-amh = <5000>;
        // Charging IC Node
        power-supplies = <&sc8886>;
        // Maximum battery voltage
        firefly,battery-max-voltage = <8400>;
        // Minimum battery voltage
        firefly,battery-min-voltage = <6000>;
        // Maximum operating temperature of the battery
        firefly,max-temp = <45>;
        // Minimum operating temperature of the battery
        firefly,min-temp = <10>;
    };
```

When using other types of batteries, you need to send the battery to the fuel gauge manufacturer to scan the parameters, otherwise the battery power measured by the fuel gauge will be inaccurate. There is also an option to contact us (sales@t-firefly.com) to assist with battery support.