# LCD
## Config Configuration

 Under AIO-3568J SDK, the default configuration file:
`kernel/arch/arm64/configs/firefly_defconfig` of EC-A3568J has already set the LCD-related configuration. If you modify it yourself, please pay attention to it. The following configuration is added:
```
CONFIG_DRM_ROCKCHIP=y
CONFIG_ROCKCHIP_DW_MIPI_DSI=y
CONFIG_DRM_PANEL_SIMPLE=y
```

## MIPI DTS Configuration

AIO-3568J SDK has MIPI DSI DTS file:`rk3568-firefly-itx-3568q-mipi101_M101014_BE45_A1.dts`, this file is a MIPI_DSI1+HDMI screen configration, MIPI_DSI0 is disabled by default, we would take the dsi1 configuration as an example to introduce the related LCD driver.

### Config VP
we can see the following statement from the DTS file:
```
&route_dsi0 {
    status = "disabled";
    connect = <&vp1_out_dsi0>;
};

&video_phy0 {
    status = "disabled";
};

&dsi0_in_vp0 {
    status = "disabled";
};

&dsi0_in_vp1 {
    status = "disabled";
};

&dsi0 {
    status = "disabled";
};

&route_dsi1 {
    status = "okay";
    connect = <&vp1_out_dsi1>;
};

&video_phy1 {
    status = "okay";
};

&dsi1_in_vp0 {
    status = "disabled";
};

&dsi1_in_vp1 {
    status = "okay";
};

&route_hdmi {
    status = "okay";
    connect = <&vp0_out_hdmi>;
};

&hdmi {
    status = "okay";
};

&hdmi_in_vp0 {
    status = "okay";
};

&hdmi_in_vp1 {
    status = "disabled";
};
```
vp means video port, this code right here saids disable DSI0, enable DSI1 using vp1, enable HDMI using vp0

DSI1 and HDMI exchange VP is also available, just make sure that different screen uses different vp

### Pin Configuration

```
/*
 * mipi_dphy1 needs to be enabled
 * when dsi1 is enabled
 */
&dsi1 {
    status = "okay";
    //rockchip,lane-rate = <1000>;
    dsi1_panel: panel@0 {
        status = "okay";
        compatible = "simple-panel-dsi";
        reg = <0>;
        backlight = <&backlight>;

        enable-gpios = <&pca9555 PCA_IO1_2 GPIO_ACTIVE_HIGH>;
        reset-gpios = <&gpio0 RK_PB0 GPIO_ACTIVE_LOW>;
        pinctrl-names = "default";
        pinctrl-0 = <&lcd_panel_reset>;
        //power-supply = <&vcc1v8_lcd0_power>;
		......

&pinctrl {
    lcd {
        lcd_panel_reset: lcd-panel-reset {
            rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
	};
};
```
The power control pins of the LCD are defined here:

|  NAME  |  GPIO  |  GPIO_ACTIVE  | 
| --- | --- |--- |
| enable-gpios(LCD_EN)  |  PCA_IO1_2  |  GPIO_ACTIVE_HIGH  |
| reset-gpios(LCD_RST) | GPIO0_B0 | GPIO_ACTIVE_LOW |

On the hardware signal, the LCD_EN is active at high level and LCD_RST is active at low levle. Please refer to the section ["GPIO Usage"](driver_gpio.html) for specific pin configuration. It should be noted that AIO-3568J dsi1's LCD_EN(PCA_IO1_2), this GPIO was from PCA9555 expansion chip, which can only be controlled when the drive is running.

### Backlight Configuration

The backlight information is configured in the DTS file, as follows:
```
    backlight: backlight {
        status = "okay";
        compatible = "pwm-backlight";
        enable-gpios = <&pca9555 PCA_IO0_6 GPIO_ACTIVE_HIGH>;
        pwms = <&pwm4 0 50000 1>;
        brightness-levels = <
             60  60  60  61  61  61  62  62
             62  63  63  63  64  64  64  65
             65  65  66  66  66  67  67  67
			......
			......
        >;
        default-brightness-level = <200>;
    };
```
* enable-gpios: backlight enable pin, active high level.
* pwms property: use to configure pwm. in the example, default used pwm4, which period is 50000ns(20KHz), and pwm is negative.
* brightness-levels property: Generally take the value 255 as a scale, so the "general brightness-levels" is an array of 256 elements. When the PWM is set to positive polarity, the back-light is positive from 0 to 255, and the duty cycle is changed from 0% to 100%. The negative polarity of 255 to 0 bits is changed from 100% to 0%. When the PWM is set to negative polarity, the opposite is also conversed.
* default-brightness-level property: the default backlight brightness, range is frome 0 to 255. 

For details, please refer to the documentation in the kernel: `kernel/Documentation/devicetree/bindings/leds/backlight/pwm-backlight.txt`
### Display Timing Configuration

#### Power On/Off Sequence

Usually, the power on/off sequence of MIPI screen is in the chapter "Power on/off sequence" of screen specifications. You can modify the dts according to the power on/off sequence of screen specifications. The panel node has the following attributes: 

```
&dsi1 {
    status = "okay";
    //rockchip,lane-rate = <1000>;
    dsi1_panel: panel@1 {
        status = "okay";
        compatible = "simple-panel-dsi";
        ...
        enable-delay-ms = <35>;
        prepare-delay-ms = <6>;
        reset-delay-ms = <25>;
        init-delay-ms = <55>;
        unprepare-delay-ms = <0>;
        disable-delay-ms = <20>;
        mipi-data-delay-ms = <200>;
        ...
    };
};
```
The property mipi-data-delay-ms is time sequence for 10.1 inches M101014-BE45-A1 MIPI Screen, you can configure this property according to the power on / off time sequence. After the MIPI is powered on or off, it needs to send initialization or exit instructions to make it work normally. The list of initialization and exit instructions is as follows:

```
&dsi1 {
	status = "okay";
	//rockchip,lane-rate = <1000>;
	dsi1_panel: panel@1 {
        ...
        panel-init-sequence = [
            //39 00 04 B9 83 10 2E
            // 15 00 02 CF FF
            05 78 01 11
            05 32 01 29
            //15 00 02 35 00
        ];

        panel-exit-sequence = [
            05 00 01 28
            05 00 01 10
        ];
		...
    };
};
```

The init-sequence and exit-sequence in the example DTS are only configured with power on/off instructions. Other initialization instructions have been burned in the IC of M101014_BE45_A1 MIPI screen. DTS does not need to configure these initialization instructions. Next, let's take a look at the implementation of power on/off timing sequence in the driver. The specific implementation is in `kernel/drivers/gpu/drm/panel/panel-simple.c`:

```
static int panel_simple_disable(struct drm_panel *panel)
{
    ...
	if (p->backlight) {
		p->backlight->props.power = FB_BLANK_POWERDOWN;
		p->backlight->props.state |= BL_CORE_FBBLANK;
		backlight_update_status(p->backlight);
	}

	if (p->desc->delay.disable)
		panel_simple_sleep(p->desc->delay.disable);

	if (p->cmd_type == CMD_TYPE_MCU) {
		err = panel_simple_xfer_mcu_cmd_seq(p, p->desc->exit_seq);
		if (err)
			dev_err(panel->dev, "failed to send exit cmds seq\n");
	}
    ...
}

static int panel_simple_unprepare(struct drm_panel *panel)
{
    ...
	if (p->desc->exit_seq) {
		if (p->dsi)
			panel_simple_xfer_dsi_cmd_seq(p, p->desc->exit_seq);
		else if (p->cmd_type == CMD_TYPE_SPI)
			err = panel_simple_xfer_spi_cmd_seq(p, p->desc->exit_seq);
		if (err)
			dev_err(panel->dev, "failed to send exit cmds seq\n");
	}

	gpiod_direction_output(p->reset_gpio, 1);
	if(!p->enable_on_always){
		gpiod_direction_output(p->enable_gpio, 0);
	}
    ...
}

static int panel_simple_prepare(struct drm_panel *panel)
{
    ...
	gpiod_direction_output(p->enable_gpio, 1);

	if (p->desc->delay.prepare)
		panel_simple_sleep(p->desc->delay.prepare);

    ...
	gpiod_direction_output(p->reset_gpio, 1);

	if (p->desc->delay.reset)
		panel_simple_sleep(p->desc->delay.reset);

	gpiod_direction_output(p->reset_gpio, 0);  //Because the driver uses an API like ``gpiod`` when controlling the reset pin timing
                                                   //so, if the DTS is configured with low level active, the reset pin will be pulled high, that is reversed
	if (p->desc->delay.init)
		panel_simple_sleep(p->desc->delay.init);

	if (p->desc->init_seq) {
		if (p->dsi)
			panel_simple_xfer_dsi_cmd_seq(p, p->desc->init_seq);
		else if (p->cmd_type == CMD_TYPE_SPI)
			err = panel_simple_xfer_spi_cmd_seq(p, p->desc->init_seq);
		if (err)
			dev_err(panel->dev, "failed to send init cmds seq\n");
	}

    if(p->desc->delay.mipi_data){
        panel_simple_sleep(p->desc->delay.mipi_data);
    }
    ...
}

static int panel_simple_enable(struct drm_panel *panel)
{
    ...
	if (p->cmd_type == CMD_TYPE_MCU) {
		err = panel_simple_xfer_mcu_cmd_seq(p, p->desc->init_seq);
		if (err)
			dev_err(panel->dev, "failed to send init cmds seq\n");
	}
	if (p->desc->delay.enable)
		panel_simple_sleep(p->desc->delay.enable);

	if (p->backlight) {
		p->backlight->props.state &= ~BL_CORE_FBBLANK;
		p->backlight->props.power = FB_BLANK_UNBLANK;
		backlight_update_status(p->backlight);
	}
    ...
}
```
For Uboot, it is implemented in `u-boot/drivers/video/drm/rockchip_panel.c`. The implementation function is in panel_simple_unprepare、panel_simple_prepare、panel_simple_enable、panel_simple_disable, the specific implementation can be viewed by yourself.

#### Display-timings

display-timings are defined in DTS:
```
        disp_timings1: display-timings {
            native-mode = <&dsi1_timing0>;
            dsi1_timing0: timing0 {
                clock-frequency = <72600000>;//<80000000>;
                hactive = <800>;//<768>;
                vactive = <1280>;
                hsync-len = <14>;   //20, 50,10
                hback-porch = <26>; //50, 56,10
                hfront-porch = <32>;//50, 30,180
	            vsync-len = <8>;//4
	            vback-porch = <20>;//4
	            vfront-porch = <80>;//8
	            hsync-active = <0>;
	            vsync-active = <0>;
	            de-active = <0>;
                pixelclk-active = <0>;
            };
        };
```
Generally, the relevant parameters can be found in the screen specification. As follows `kernel/drivers/gpu/drm/panel/panel-simple.c`,  `panel_simple_probe` initializes the function that gets the timing sequence.
```
static int panel_simple_probe(struct device *dev, const struct panel_desc *desc){
...
 panel->base.funcs = &panel_simple_funcs;
...
}
...
static const struct drm_panel_funcs panel_simple_funcs = {
	.loader_protect = panel_simple_loader_protect,
	.disable = panel_simple_disable,
	.unprepare = panel_simple_unprepare,
	.prepare = panel_simple_prepare,
	.enable = panel_simple_enable,
	.get_modes = panel_simple_get_modes,
	.get_timings = panel_simple_get_timings,
};
```

`panel_simple_get_timings` was defined in `kernel/drivers/gpu/drm/panel/panel-simple.c`:
```
static int panel_simple_get_timings(struct drm_panel *panel, unsigned int num_timings, struct display_timing *timings)
{
	struct panel_simple *p = to_panel_simple(panel);
	unsigned int i;

	if (p->desc->num_timings < num_timings)
		num_timings = p->desc->num_timings;

	if (timings)
		for (i = 0; i < num_timings; i++)
			timings[i] = p->desc->timings[i];

	return p->desc->num_timings;
}
```
For details, please refer to the following attachments:
[Rockchip DRM Panel Porting Guide.pdf](http://download.t-firefly.com/product/Board/Common/Document/Developer/Rockchip_Developer_Guide_DRM_Panel_Porting_CN.pdf)