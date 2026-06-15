# LCD使用
## 简介

ROC-RK3399-PC 开发板默认外置支持了两个LCD屏接口，一个是MIPI-DSI，一个是eDP，板子上对应的LCD接口位置如下图所示：

![](../../../rk3399_img/AIO-3399Pro-JD4/roc-rk3399-pc6.jpg)

* MIPI 接口
* eDP 接口
* 通用type-C转DP 接口
* HDMI 接口

**ROC-RK3399-PC 的ubuntu系统默认支持MIPI屏幕，如果使用eDP屏幕需要更换DTS。**


## MIPI屏

### 配置MIPI-DSI

ROC-RK3399-PC 的MIPI-DSI的使用的是RK原厂的基本配置，如果不考虑热拔插和功耗控制可以不配置其它管脚：
```
kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi：

dsi: dsi@ff960000 {
	compatible = "rockchip,rk3399-mipi-dsi";
	reg = <0x0 0xff960000 0x0 0x8000>;
	interrupts = <GIC_SPI 45 IRQ_TYPE_LEVEL_HIGH 0>;
	clocks = <&cru SCLK_DPHY_PLL>, <&cru PCLK_MIPI_DSI0>,
		 <&cru SCLK_DPHY_TX0_CFG>;
	clock-names = "ref", "pclk", "phy_cfg";
	power-domains = <&power RK3399_PD_VIO>;
	resets = <&cru SRST_P_MIPI_DSI0>;
	reset-names = "apb";
	rockchip,grf = <&grf>;
	status = "disabled";
	#address-cells = <1>;
	#size-cells = <0>;

	ports {
		port {
			#address-cells = <1>;
			#size-cells = <0>;

			dsi_in_vopb: endpoint@0 { //从RK自定义显卡vopb输入
				reg = <0>;
				remote-endpoint = <&vopb_out_dsi>;
			};

			dsi_in_vopl: endpoint@1 {//从RK自定义显卡vopl输入
				reg = <1>;
				remote-endpoint = <&vopl_out_dsi>;
			};
		};
	};
};
```

其它相应的更改如下：
```
kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-mipi.dts

//mipi
&dsi {
	status = "okay";
	//status = "okay";
	dsi_panel: panel {
		compatible ="simple-panel-dsi";
		reg = <0>;
		backlight = <&backlight>;
		power-supply = <&vcc_lcd>;
		dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST)>;
		dsi,format = <MIPI_DSI_FMT_RGB888>;
		bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;
		dsi,lanes = <4>;
		reset-delay-ms = <20>;
		init-delay-ms = <20>;
		enable-delay-ms = <120>;
		prepare-delay-ms = <120>;
		status = "okay";

		panel-init-sequence = [
			05 20 01 29
			05 96 01 11
		];

		panel-exit-sequence = [
			05 05 01 28
			05 78 01 10
		];

		disp_timings: display-timings {
			native-mode = <&timing0>;

			timing0: timing0 {
				clock-frequency = <80000000>;
				hactive = <768>;
				vactive = <1024>;
				hsync-len = <20>;
				hback-porch = <110>;
				hfront-porch = <170>;
				vsync-len = <40>;
				vback-porch = <130>;
				vfront-porch = <136>;
				hsync-active = <0>;
				vsync-active = <0>;
				de-active = <0>;
				pixelclk-active = <0>;
			};
		};
	};
};
```

### 配置背光

ROC-RK3399-PC 开发板外置了一个背光接口用来控制屏幕背光，如下图所示：

![](../../../rk3399_img/AIO-3399Pro-JD4/MIPI-DSI_pin.jpg)

在DTS文件：kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc.dtsi中配置了背光信息，brightness-levels属性：配置背光亮度数组，最大值为255，配置暗区和亮区，并把亮区数组做255的比例调节。比如范例中暗区是255-221，亮区是220-0（具体请参考kernel中的说明文档：kernel/Documentation/devicetree/bindings/leds/backlight/pwm-backlight.txt）。其中default-brightness-level属性设置开机时默认背光亮度，范围为0-255。基本配置信息如下：
```
/ {
	compatible = "firefly,roc-rk3399-pc", "rockchip,rk3399";

	backlight: backlight {
		status = "okay";
		compatible = "pwm-backlight";
		pwms = <&pwm1 0 25000 0>;
		brightness-levels = <
			  0   1   2   3   4   5   6   7
			  8   9  10  11  12  13  14  15
			 16  17  18  19  20  21  22  23
			 24  25  26  27  28  29  30  31
			 32  33  34  35  36  37  38  39
			 40  41  42  43  44  45  46  47
			 48  49  50  51  52  53  54  55
			 56  57  58  59  60  61  62  63
			 64  65  66  67  68  69  70  71
			 72  73  74  75  76  77  78  79
			 80  81  82  83  84  85  86  87
			 88  89  90  91  92  93  94  95
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
			248 249 250 251 252 253 254 255>;
		default-brightness-level = <200>;
	};
  ...
}
```
pwms属性：配置PWM，范例里面默认使用pwm1，25000ns是周期(40 KHz)：
```
pwms = <&pwm1 0 25000 0>;
```

### 配置时序
* MIPI屏的 Timing 写在DTS文件中，在kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-mipi.dts中可以看到以下语句:
```
disp_timings: display-timings {
    native-mode = <&timing0>;
    timing0: timing0 {
         clock-frequency = <80000000>;
         hactive = <768>;
         vactive = <1024>;
         hsync-len = <20>;   //20, 50
         hback-porch = <130>; //50, 56
         hfront-porch = <150>;//50, 30
         vsync-len = <40>;
         vback-porch = <130>;
         vfront-porch = <136>;
         hsync-active = <0>;
         vsync-active = <0>;
         de-active = <0>;
         pixelclk-active = <0>;
           };
    }
}
```

* Kernel
在kernel/drivers/gpu/drm/panel/panel-simple.c中可以看到在初始化函数panel_simple_probe中初始化了获取时序的函数。

```
static int panel_simple_probe(struct device *dev, const struct panel_desc *desc){
···
 panel->base.funcs = &panel_simple_funcs;
···
}
```

该函数的在kernel/drivers/gpu/drm/panel/panel-simple.c中也有定义：

```
static int panel_simple_get_timings(struct drm_panel *panel,unsigned int num_timings,struct display_timing *timings)
{
    struct panel_simple *p = to_panel_simple(panel);
    unsigned int i;
    if (!p->desc)
        return 0;

    if (p->desc->num_timings < num_timings)
        num_timings = p->desc->num_timings;

    if (timings)
        for (i = 0; i < num_timings; i++)
        timings[i] = p->desc->timings[i];
    return p->desc->num_timings;
}
```
## eDP

待续。。