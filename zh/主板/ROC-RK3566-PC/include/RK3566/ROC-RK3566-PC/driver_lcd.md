# LCD使用
## Config配置

ROC-RK3566-PC的SDK中的`kernel/arch/arm64/configs/firefly_defconfig`文件已经把LCD相关的配置设置好了，如果自己做了修改，请注意把以下配置加上：
```
CONFIG_DRM_ROCKCHIP=y
CONFIG_ROCKCHIP_DW_MIPI_DSI=y
CONFIG_DRM_PANEL_SIMPLE=y
```
## MIPI DTS配置

ROC-RK3566-PC的SDK有MIPI DSI的DTS文件：`kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc-mipi101_M101014_BE45_A1.dts`，接下来以此DTS中的dsi0配置作为范例来进行相关的LCD驱动介绍。

### 引脚配置

从DTS文件中我们可以看到以下语句：
```
&dsi0 {
	status = "okay";
	rockchip,lane-rate = <500>;
	dsi0_panel: panel@0 {
	...          
        reset-gpios = <&gpio0 RK_PC2 GPIO_ACTIVE_LOW>;  
	...
        enable-gpios = <&gpio0 RK_PC7 GPIO_ACTIVE_HIGH>; 
	...
};
```
这里定义了LCD的控制引脚：

|  NAME  |  GPIO  |  GPIO_ACTIVE  | 
| --- | --- |--- |
| reset-gpios  |  GPIO0_C2  |  GPIO_ACTIVE_LOW  |
| enable-gpios | GPIO0_C7 | GPIO_ACTIVE_HIGH |

在硬件信号上enable引脚高电平有效，而reset引脚是低电平有效。  

具体的引脚配置请参考[《GPIO 使用》](driver_gpio.html)一节。
### 背光配置

在DTS文件中配置了背光信息，如下：
```
backlight:backlight {
		status = "okay";
		compatible = "pwm-backlight";
		pwms = <&pwm4 0 50000 1>;
		brightness-levels = <
		     60  60  60  61  61  61  62  62
		     62  63  63  63  64  64  64  65
		     65  65  66  66  66  67  67  67
		     68  68  68  69  69  69  70  70
		     70  71  71  71  72  72  72  73
		     73  73  74  74  74  75  75  75
		     76  76  76  77  77  77  78  78
		     78  79  79  79  80  80  80  81
		     81  81  82  82  82  83  83  83
		     84  84  84  85  85  85  86  86
		     86  87  87  87  88  88  88  89
		     89  89  90  91  92  93  94  95
		     96  97  98  99 100 101 102 103
		    104 105 106 107 108 109 110 111
		    112 113 114 115 116 117 118 119
		    120 121 122 123 124 125 126 127
		    128 129 130 131 132 133 134 135
		    136 137 138 139 140 141 142 143
		    144 145 146 147 148 149 150 151
		    152 153 154 155 156 157 158 159
		    160 161 162 163 164 165 166 167
		    168 169 170 171 172 173 174 175
		    176 177 178 179 180 181 182 183
		    184 185 186 187 188 189 190 191
		    192 193 194 195 196 197 198 199
		    200 201 202 203 204 205 206 207
		    208 209 210 211 212 213 214 215
		    216 217 218 219 220 221 222 223
		    224 225 226 227 228 229 230 231
		    232 233 234 235 236 237 238 239
		    240 241 242 243 244 245 246 247
		    248 249 250 251 252 253 254 255
               >;
               default-brightness-level = <200>;
        	enable-gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
	};
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
&dsi0 {
    status = "okay";
    //rockchip,lane-rate = <500>;
    dsi0_panel: panel@0 {
        status = "okay";
        compatible = "simple-panel-dsi";
        ...
        enable-delay-ms = <35>;
        prepare-delay-ms = <6>;
        reset-delay-ms = <25>;
        init-delay-ms = <55>;
        unprepare-delay-ms = <0>;
        disable-delay-ms = <20>;
        size,width = <120>;
        size,height = <170>;
        dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;
        dsi,format = <MIPI_DSI_FMT_RGB888>;
        dsi,lanes  = <4>;

        ...
    };
};
```
MIPI上下电后需要发送初始化或退出指令才能使之正常工作，初始化和退出指令列表可以参考`kernel/arch/arm64/boot/dts/rockchip/rk3566-roc-pc-mipi101_M101014_BE45_A1.dts`，列表如下：
```
&dsi0 {
	status = "okay";
	//rockchip,lane-rate = <500>;
	dsi0_panel: panel@0 {
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
	}
};
```
接下来我们来看看上下电时序在驱动中的实现，
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
display-timings {
            native-mode = <&timing1>;
            timing1: timing1 {
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

}
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


