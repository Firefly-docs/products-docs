# LCD使用
## Config配置

AIO-3566JD4的SDK下`kernel/arch/arm64/configs/firefly_defconfig`已经把LCD相关的配置设置好了，如果自己做了修改，请注意把以下配置加上：
```
CONFIG_DRM_ROCKCHIP=y
CONFIG_ROCKCHIP_DW_MIPI_DSI=y
CONFIG_DRM_PANEL_SIMPLE=y
```
## MIPI DTS配置

AIO-3566JD4的SDK有MIPI DSI的DTS文件：
`rk3568-firefly-itx-3568q-mipi101_M101014_BE45_A1.dts`，此 DTS 文件为 MIPI_DSI1+HDMI 屏配置，MIPI_DSI0 默认关闭，接下来以此 DTS 中的 DSI1 配置作为范例来进行相关的 LCD 驱动介绍。

### VP 分配

从DTS文件中我们可以看到以下语句：
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
vp 是视频接口的意思，这一段表示 DSI0 关闭，DSI1 开启并使用 vp1，HDMI 开启并使用 vp0

DSI1 和 HDMI 交换 vp 也是可以的，要保证不同的屏幕需要使用不同的 vp

### 引脚配置

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
这里定义了LCD的控制引脚：

|  NAME  |  GPIO  |  GPIO_ACTIVE  | 
| --- | --- |--- |
| enable-gpios(LCD_EN)  |  PCA_IO1_2  |  GPIO_ACTIVE_HIGH  |
| reset-gpios(LCD_RST) | GPIO0_B0 | GPIO_ACTIVE_LOW |

在硬件信号上LCD_EN引脚高电平有效，而LCD_RST引脚是低电平有效，具体的引脚配置请参考[《GPIO 使用》](driver_gpio.html)一节。需要注意的是AIO-3566JD4的dsi1中，LCD_EN:(PCA_IO1_2)使用的是PCA9555扩展芯片引出来的gpio，需要PCA9555 驱动起来以后才能控制该gpio。
### 背光配置

在DTS文件中配置了背光信息，如下：
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
* enable-gpios：背光使能脚，高电平有效。
* pwms属性：配置PWM，范例里面默认使用pwm4，50000ns是周期(20 KHz)，pwm为负极性。
* brightness-levels属性：配置背光亮度数组，一般以值 255 为一个 scale，当 PWM 设置为正极性时，从 0~255 表示背光为正极，占空比从 0%~100% 变化，255~0 为负极性，占空比从 100%~0% 变化；当 PWM 设置为负极性时，反之。
* default-brightness-level属性：开机时默认背光亮度，范围为0-255。

具体请参考kernel中的说明文档：`kernel/Documentation/devicetree/bindings/leds/backlight/pwm-backlight.txt`
### 显示时序配置

#### Power on/off sequence

MIPI屏的上下电时序通常都在规格书的`Power on/off sequence`章节中，可根据规格书的上下电时序修改dts，panel节点中有如下属性:
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
mipi-data-delay-ms为10.1寸 M101014-BE45-A1 MIPI屏所需时序，调试屏幕时可根据本身上下电时序是否需要再配置此属性。MIPI上下电后需要发送初始化或退出指令才能使之正常工作，列表如下：
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
范例 dts 中的 init-sequence 和 panel-exit-sequence，只是配置了上电和下电指令, 其他的初始化指令已经烧录在 M101014-BE45-A1 这款MIPI屏的 IC 里，dts 无需配置。接下来我们来看看上下电时序在驱动中的实现，
具体实现在`kernel/drivers/gpu/drm/panel/panel-simple.c`中：
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

	gpiod_direction_output(p->reset_gpio, 0);  //由于驱动在控制reset脚时序的时候使用了`gpiod`这类的API
                                                   //如果在DTS配置了低电平有效，那么这里就会将reset引脚拉高，即取反。
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
uboot实现在`u-boot/drivers/video/drm/rockchip_panel.c`中的panel_simple_unprepare、panel_simple_prepare、panel_simple_enable
、panel_simple_disable 4个函数中，具体实现可以自己查看。

#### Display-Timings

display_timings在dts中定义：

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
相关的参数一般可以在屏的规格书中找到，如下在`kernel/drivers/gpu/drm/panel/panel-simple.c`的panel_simple_probe中初始化了获取时序的函数。
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
...
```
该函数在`kernel/drivers/gpu/drm/panel/panel-simple.c`中定义：
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
详细说明可参考以下附件：
[Rockchip DRM Panel Porting Guide.pdf](http://download.t-firefly.com/product/Board/Common/Document/Developer/Rockchip_Developer_Guide_DRM_Panel_Porting_CN.pdf)
