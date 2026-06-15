# LCD使用
## 简介

ROC-RK3328-PC开发板默认外置支持了两个LCD屏接口，一个是LVDS，一个是EDP，接口对应板子上的位置如下图：

![](../../../rk3399_img/ROC-RK3328-PC/lcd_interface.jpg)

另外板子也支持MIPI屏幕，但需要注意的是MIPI和LVDS是复用的，使用MIPI之后不能使用LVDS。MIPI接口如下图:
![](../../../rk3399_img/ROC-RK3328-PC/lcd_mipi.jpg)

## LVDS屏
### DTS配置
#### 引脚配置

ROC-RK3328-PC的SDK有LVDS DSI的DTS文件：kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4-lvds-HSX101H40C.dts，从该文件中我们可以看到以下语句：
```
/ {
        model = "AIO-3399Pro-JD4 10.1 inches LVDS HSX101H40C (Android)";
        compatible = "rockchip,rk3399pro-firefly-aiojd4-lvds", "rockchip,rk3399pro";

};

&backlight {
                status = "okay";
                pwms = <&pwm0 0 25000 1>;
                brightness-levels = <
                         /*0 20  20  21  21  22  22  23
                         23  24  24  25  25  26  26  27
                         27  28  28  29  29  30  30  31
                         31  32  32  33  33  34  34  35
                         35  36  36  37  37  38  38  39
                         40*/41  42  43  44  45  46  47
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
                        248 249 250 251 252 253 254 255
                >;
                default-brightness-level = <200>;
                enable-gpios = <&gpio1 RK_PC5 GPIO_ACTIVE_HIGH>;
};

&dsi {
        status = "okay";
        rockchip,lane-rate = <864>;
        panel@0 {
                compatible ="simple-panel-dsi";
                reg = <0>;
                backlight = <&backlight>;

                enable-5v-gpios = <&gpio4 RK_PD6 GPIO_ACTIVE_HIGH>;
                enable-tc-gpios = <&gpio1 RK_PA2 GPIO_ACTIVE_HIGH>;
                reset-gpios = <&gpio4 RK_PD1 GPIO_ACTIVE_LOW>;


                dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;
                dsi,format = <MIPI_DSI_FMT_RGB888>;
                bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;
                dsi,lanes = <4>;

        dsi,channel = <0>;

        enable-delay-ms = <35>;
        prepare-delay-ms = <6>;

        unprepare-delay-ms = <0>;
        disable-delay-ms = <20>;

        size,width = <120>;
        size,height = <170>;
        status = "okay";
        panel-init-sequence = [
                29 02 06 3C 01 09 00 07 00
                29 02 06 14 01 06 00 00 00
                29 02 06 64 01 0B 00 00 00
                29 02 06 68 01 0B 00 00 00
                29 02 06 6C 01 0B 00 00 00
                29 02 06 70 01 0B 00 00 00
                29 02 06 34 01 1F 00 00 00
                29 02 06 10 02 1F 00 00 00
                29 02 06 04 01 01 00 00 00
                29 02 06 04 02 01 00 00 00
                29 02 06 50 04 30 01 F0 03       //framesync mode
                29 02 06 54 04 0A 00 32 00      //hsync_len = 0x0A    hback_porch =0x32
                29 02 06 58 04 20 03 32 00      //hactive   = 0x320   hfront_porch=0x32
                29 02 06 5C 04 12 00 0A 00      //vsync_len = 0x12    vback_porch =0x0A
                29 02 06 60 04 00 05 0A 00      //vactive   = 0x500   vfront_porch=0x0A

                29 02 06 64 04 01 00 00 00
                29 02 06 A0 04 06 80 44 00
                29 02 06 A0 04 06 80 04 00
                29 02 06 04 05 04 00 00 00
                29 02 06 80 04 00 01 02 03
                29 02 06 84 04 04 07 05 08
                29 02 06 88 04 09 0A 0E 0F
                29 02 06 8C 04 0B 0C 0D 10
                29 02 06 90 04 16 17 11 12
                29 02 06 94 04 13 14 15 1B
                29 02 06 98 04 18 19 1A 06
                29 02 06 9C 04 31 04 00 00
        ];

                panel-exit-sequence = [
                        05 05 01 28
                        05 78 01 10
                ];

                display-timings {
                        native-mode = <&timing0>;
                        timing0: timing0 {
                                clock-frequency = <72000000>; //@60
                                hactive = <800>;
                                vactive = <1280>;
                                hsync-len = <10>;
                                hback-porch = <50>;
                                hfront-porch = <50>;
                                vsync-len = <18>;
                                vback-porch = <10>;
                                vfront-porch = <10>;
                                hsync-active = <0>;
                                vsync-active = <0>;
                                de-active = <0>;
                                pixelclk-active = <0>;
                        };
                };
        };
};

```
这里定义了LCD的电源控制引脚：

```
enable-gpios:(GPIO1_C5)GPIO_ACTIVE_HIGH
enable-5v-gpios:(GPIO4_D6)GPIO_ACTIVE_HIGH
enable-tc-gpios:(GPIO1_A2)GPIO_ACTIVE_HIGH
reset-gpios:(GPIO4_D1)GPIO_ACTIVE_HIGH
```
都是高电平有效，具体的引脚配置请参考[《GPIO 使用》](driver_gpio.md)一节。
#### LVDS配置背光

ROC-RK3328-PC开发板外置了一个背光接口用来控制屏幕背光，如下图所示：

![](../../../rk3399_img/ROC-RK3328-PC/lcd_back_light.jpg)

```
/ {
    compatible = "rockchip,rk3399-firefly-core", "rockchip,rk3399";

    backlight: backlight {
        status = "disabled";
        compatible = "pwm-backlight";
        pwms = <&pwm0 0 25000 0>;
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
```
pwms属性：配置PWM，范例里面默认使用pwm0，25000ns是周期(40 KHz)。LVDS需要加背光电源控制脚，在kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4-lvds-HSX101H40C.dts中可以看到以下语句：
```
&backlight {
     status = "okay";
     pwms = <&pwm0 0 25000 1>;
     brightness-levels = <
              /*0 20  20  21  21  22  22  23
              23  24  24  25  25  26  26  27
              27  28  28  29  29  30  30  31
              31  32  32  33  33  34  34  35
              35  36  36  37  37  38  38  39
              40*/41  42  43  44  45  46  47
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
             248 249 250 251 252 253 254 255
     >;
};
```
因此使用时需修改DTS文件。

brightness-levels属性：配置背光亮度数组，最大值为255，配置暗区和亮区，并把亮区数组做255的比例调节。比如范例中暗区是255-221，亮区是220-0。
default-brightness-level属性：开机时默认背光亮度，范围为0-255。
具体请参考kernel中的说明文档：kernel/Documentation/devicetree/bindings/leds/backlight/pwm-backlight.txt
#### LVDS配置显示时序

与EDP屏不同，LVDS屏的 Timing 写在DTS文件中，在kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4-lvds-HSX101H40C.dts中可以看到以下语句：

```
display-timings {
        native-mode = <&timing0>;
        timing0: timing0 {
                clock-frequency = <72000000>;
                hactive = <800>;
                vactive = <1280>;
                hsync-len = <10>;
                hback-porch = <50>;
                hfront-porch = <50>;
                vsync-len = <18>;
                vback-porch = <10>;
                vfront-porch = <10>;
                hsync-active = <0>;
                vsync-active = <0>;
                de-active = <0>;
                pixelclk-active = <0>;
        };
};
```

时序属性参考下图：

![](../../../rk3399_img/lcd_sequence.jpg)

#### Init Code
lvds屏上完电后需要发送初始化指令才能使之工作。初始化指令需要以下工具文档生成，下载[TC358764_5_774_5XBG_DSI-LVDS_Tv11p_nm_1280x800.xls](https://www.t-firefly.com/share/index/index/id/5b0d81aee388780c43b1af9784845986.html)

##### 如何配置LVDS panel-init-sequence
以1280x800单lvds为例:
首先打开TC358764_5_774_5XBG_DSI-LVDS_Tv11p_nm_1280x800.xls
![](../../../rk3399_img/page.png)
选择页面"Timing Parameters_SYNC_EVENT"，按照LVDS屏的时序填入LVDS timing黄色单元,一般只需填入以下单元即可。
* HPW / HBPR / HDISPR / HFPR 分别对应 hsync-len / hback-porch / hactive / hfront-porch
* VPW / VBPR / VDISPR / VFPR 分别对应 vhsync-len / vback-porch / vactive / vfront-porch

LVDS timing填入完成后还需配置常规参数
![](../../../rk3399_img/parameter.png)
* 1.根据LVDS屏规格书确认LVDS Link和LVDS output format并选择屏的参数。
* 2.计算LVDS clock(蓝色单元无法写入，需要黄色单元自动计算得出)，需要填入DSI Clock(HOST), Pixel Clock Source, Pixel Clock Divider。计算公式如下:DSI Clock/Pixel Clock Source/Pixel Clock Divider=LVDS Clock

填入上述黄色单元基本上完成配置，接下来选择页面"Source"即可看到转换后的Comment
![](../../../rk3399_img/source.png)
以上面为例"013C 00030005"，mipi command就应该是"29 02 06 3C 01 05 00 03 00"
* 29 : packet ID
* 02 : 2ms delay
* 06 : 6 bytes
* 3C 01 : address = 0x013C
* 00 03 00 05 : data=0x05000300

将页面source所有地址写入数据，即可完成初始化指令panel-init-sequence。


dts在kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4-lvds-HSX101H40C.dts中可以看到lvds的初始化指令列表：
```
&dsi {
   status = "okay";
        ...
  panel-init-sequence = [
  29 00 06 3C 01 09 00 07 00
  29 00 06 14 01 06 00 00 00
  29 00 06 64 01 0B 00 00 00
  29 00 06 68 01 0B 00 00 00
  29 00 06 6C 01 0B 00 00 00
  29 00 06 70 01 0B 00 00 00
  29 00 06 34 01 1F 00 00 00
  29 00 06 10 02 1F 00 00 00
  29 00 06 04 01 01 00 00 00
  29 00 06 04 02 01 00 00 00
  29 00 06 50 04 20 01 F0 03
  29 00 06 54 04 32 00 B4 00
  29 00 06 58 04 80 07 48 00
  29 00 06 5C 04 0A 00 19 00
  29 00 06 60 04 38 04 0A 00
  29 00 06 64 04 01 00 00 00
  29 01 06 A0 04 06 80 44 00
  29 00 06 A0 04 06 80 04 00
  29 00 06 04 05 04 00 00 00
  29 00 06 80 04 00 01 02 03
  29 00 06 84 04 04 07 05 08
  29 00 06 88 04 09 0A 0E 0F
  29 00 06 8C 04 0B 0C 0D 10
  29 00 06 90 04 16 17 11 12
  29 00 06 94 04 13 14 15 1B
  29 00 06 98 04 18 19 1A 06
  29 02 06 9C 04 33 04 00 00
  ];
  panel-exit-sequence = [
                        05 05 01 28
                        05 78 01 10
  ];
        ...
};
```
命令格式以及说明可参考以下附件：
[Rockchip DRM Panel Porting Guide.pdf](http://www.t-firefly.com/ueditor/php/upload/file/20171213/1513128959299913.pdf)

* kernel
发送指令可以看到在kernel/drivers/gpu/drm/panel/panel-simple.c文件中的操作：
```
static int panel_simple_enable(struct drm_panel *panel)
{
    struct panel_simple *p = to_panel_simple(panel);
    int err;
    if (p->enabled)
        return 0;
    DBG("enter\n");
    if (p->on_cmds) {
        err = panel_simple_dsi_send_cmds(p, p->on_cmds);
        if (err)
            dev_err(p->dev, "failed to send on cmds\n");
    }
    if (p->desc && p->desc->delay.enable) {
        DBG("p->desc->delay.enable=%d\n", p->desc->delay.enable);
        msleep(p->desc->delay.enable);
    }
    if (p->backlight) {
        DBG("open backlight\n");
        p->backlight->props.power = FB_BLANK_UNBLANK;
        backlight_update_status(p->backlight);
    }
    p->enabled = true;
    return 0;
}
```
* u-boot
发送指令可以看到在u-boot/drivers/video/rockchip-dw-mipi-dsi.c文件中的操作：
```
static int rockchip_dw_mipi_dsi_enable(struct display_state *state)
{
    struct connector_state *conn_state = &state->conn_state;
    struct crtc_state *crtc_state = &state->crtc_state;
    const struct rockchip_connector *connector = conn_state->connector;
    const struct dw_mipi_dsi_plat_data *pdata = connector->data;
    struct dw_mipi_dsi *dsi = conn_state->private;
    u32 val;
    DBG("enter\n");
    dw_mipi_dsi_set_mode(dsi, DW_MIPI_DSI_VID_MODE);
    dsi_write(dsi, DSI_MODE_CFG, ENABLE_CMD_MODE);
    dw_mipi_dsi_set_mode(dsi, DW_MIPI_DSI_VID_MODE);
    if (!pdata->has_vop_sel)
        return 0;
    if (pdata->grf_switch_reg) {
        if (crtc_state->crtc_id)
            val = pdata->dsi0_en_bit | (pdata->dsi0_en_bit << 16);
        else
            val = pdata->dsi0_en_bit << 16;
        writel(val, RKIO_GRF_PHYS + pdata->grf_switch_reg);
    }
    debug("vop %s output to dsi0\n", (crtc_state->crtc_id) ? "LIT" : "BIG");
    //rockchip_dw_mipi_dsi_read_allregs(dsi);
    return 0;
}
```
#### 常见问题

##### 1. 画面抖动闪屏

排查屏参数是否超出屏规格书限定周期，排查屏时钟大小。"Timing Parameters_SYNC_EVENT"所有参数变动必须和comment同步调整。

##### 2. 颜色显示异常

尝试同步调整color mapping或者lvds timing。

**NOTE**: 页面"How to use"有详细步骤，其他参数说明可以参考文档"页面"菜单。

## EDP屏
### DTS配置
#### 引脚配置

ROC-RK3328-PC的SDK有EDP DSI的DTS文件：kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4-edp.dts，从该文件中我们可以看到以下语句：
```
  edp_panel: edp-panel {
		/* config 2 */
		compatible = "lg,lp079qx1-sp0v";
		/* config 3 */
		//compatible = "simple-panel";
		bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;

		backlight = <&backlight>;

		ports {
			panel_in_edp: endpoint {
				remote-endpoint = <&edp_out_panel>;
			};
		};

		power_ctr: power_ctr {
		power_enable = <1>;
               rockchip,debug = <0>;
               lcd_en: lcd-en {
                       gpios = <&gpio1 4 GPIO_ACTIVE_HIGH>;
					   pinctrl-names = "default";
					   pinctrl-0 = <&lcd_panel_enable>;
                       rockchip,delay = <20>;
               };
               lcd_pwr_en: lcd-pwr-en {
                       gpios = <&gpio0 1 GPIO_ACTIVE_HIGH>;
                       pinctrl-names = "default";
                       pinctrl-0 = <&lcd_panel_pwr_en>;
                       rockchip,delay = <10>;
               };
       };
};
···
&pinctrl {
	lcd-panel {
		lcd_panel_enable: lcd-panel-enable {
			rockchip,pins = <1 4 RK_FUNC_GPIO &pcfg_pull_up>;
		};
        lcd_panel_pwr_en: lcd-panel-pwr-en {
            rockchip,pins = <0 1 RK_FUNC_GPIO &pcfg_pull_up>;
        };
	};
};

```
这里定义了LCD的电源控制引脚：
```
lcd_en:(GPIO1_A4)GPIO_ACTIVE_HIGH
lcd_pwr_en:(GPIO0_A1)GPIO_ACTIVE_HIGH
```
都是高电平有效，具体的引脚配置请参考[《GPIO 使用》](driver_gpio.md)一节。

#### EDP配置背光
因为背光接口是公用的，所以可参考上述LVDS的配置方法。

#### EDP配置显示时序
kernel 把 Timing 写在 panel-simple.c 中， 直接以短字符串匹配 在drivers/gpu/drm/panel/panel-simple.c文件中有以下语句

```
static const struct drm_display_mode lg_lp079qx1_sp0v_mode = {
  .clock = 200000,
  .hdisplay = 1536,
  .hsync_start = 1536 + 12,
  .hsync_end = 1536 + 12 + 16,
  .htotal = 1536 + 12 + 16 + 48,
  .vdisplay = 2048,
  .vsync_start = 2048 + 8,
  .vsync_end = 2048 + 8 + 4,
  .vtotal = 2048 + 8 + 4 + 8,
  .vrefresh = 60,
  .flags = DRM_MODE_FLAG_NVSYNC | DRM_MODE_FLAG_NHSYNC,
};

static const struct panel_desc lg_lp097qx1_spa1 = {
  .modes = &lg_lp097qx1_spa1_mode,
  .num_modes = 1,
  .size = {
    .width = 320,
    .height = 187,
  },
};

... ...

   static const struct of_device_id platform_of_match[] = {
  {
    .compatible = "simple-panel",
    .data = NULL,
  },{

  }, {
    .compatible = "lg,lp079qx1-sp0v",
    .data = &lg_lp079qx1_sp0v,
  }, {

  }, {
    /* sentinel */
  }
};
```

MODULE_DEVICE_TABLE(of, platform_of_match); 时序的参数在结构体lg_lp079qx1_sp0v_mode中配置。

* U-boot

把 Timing 写在 rockchip_panel.c 中， 直接以短字符串匹配 在drivers/video/rockchip_panel.c文件中有以下语句：
```
static const struct drm_display_mode lg_lp079qx1_sp0v_mode = {
  .clock = 200000,
  .hdisplay = 1536,
  .hsync_start = 1536 + 12,
  .hsync_end = 1536 + 12 + 16,
  .htotal = 1536 + 12 + 16 + 48,
  .vdisplay = 2048,
  .vsync_start = 2048 + 8,
  .vsync_end = 2048 + 8 + 4,
  .vtotal = 2048 + 8 + 4 + 8,
  .vrefresh = 60,
  .flags = DRM_MODE_FLAG_NVSYNC | DRM_MODE_FLAG_NHSYNC,
};
static const struct rockchip_panel g_panel[] = {
  {
    .compatible = "lg,lp079qx1-sp0v",
    .mode = &lg_lp079qx1_sp0v_mode,
  }, {
    .compatible = "auo,b125han03",
    .mode = &auo_b125han03_mode,
  },
};
```
时序的参数在结构体lg_lp079qx1_sp0v_mode中配置。

## MIPI屏
客户根据需要在自行添加mipi硬件接口之后，配置MIPI屏的 Timing dts文件，在kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-aiojd4-mipi.dts中可以看到以下语句：
```
&dsi {
        status = "okay";
        rockchip,lane-rate = <864>;
        panel@0 {
                compatible ="simple-panel-dsi";
                reg = <0>;
                backlight = <&backlight>;

                enable-5v-gpios = <&gpio4 RK_PD6 GPIO_ACTIVE_HIGH>; //vcc_5v_en
                enable-tc-gpios = <&gpio1 RK_PA2 GPIO_ACTIVE_HIGH>; //tc358775 power
                reset-gpios = <&gpio4 RK_PD1 GPIO_ACTIVE_LOW>; //tc358775 reset


                dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;
                dsi,format = <MIPI_DSI_FMT_RGB888>;
                bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;
                dsi,lanes = <4>;

        dsi,channel = <0>;

        enable-delay-ms = <35>;
        prepare-delay-ms = <6>;

        unprepare-delay-ms = <0>;
        disable-delay-ms = <20>;

        size,width = <120>;
        size,height = <170>;
        status = "okay";
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
                display-timings {
                        native-mode = <&timing0>;
                        timing0: timing0 {
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
        };
};
```

Kernel
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

mipi屏上完电后需要发送初始化指令才能使之工作，可以在kernel/arch/arm64/boot/dts/rockchip/rk3399pro-firefly-mipi.dts中可以看到mipi的初始化指令列表：
```
&mipi_dsi {
            status = "okay";
        ...
            panel-init-sequence = [
                05 20 01 29
                05 96 01 11
            ];

            panel-exit-sequence = [
                05 05 01 28
                05 78 01 10
            ];
        ...
};
```

发送指令同样在kernel/drivers/gpu/drm/panel/panel-simple.c文件中：
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
命令格式以及说明可参考以下附件：
[Rockchip DRM Panel Porting Guide.pdf](http://www.t-firefly.com/ueditor/php/upload/file/20171213/1513128959299913.pdf)