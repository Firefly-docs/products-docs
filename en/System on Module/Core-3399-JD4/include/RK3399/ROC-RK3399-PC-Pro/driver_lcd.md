# LCD
## Introduction

Core-3399-JD4 development board supports two LCD screen interfaces by default, one is MIPI (Maximum resolution support 2560x1600@60fps), the other is EDP(Maximum resolution support 2K@60fps). The positions of interfaces on the board are shown as follows:

* 10.1 in. MIPI
    ![](../../../rk3399_img/Core-3399-JD4/panel_mipi101.jpg)


* 10.1 in. EDP
    ![](../../../rk3399_img/Core-3399-JD4/panel_edp101.jpg)


## MIPI Driver Configuration
### Pin Configuration
For pin configuration, refer to the File：`kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1.dts`
#### Chip Power on Configuration
From the file `rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1.dts` we can see the following statement:

```
&dsi {
    ⋯
    power_ctr: power_ctr {
            rockchip,debug = <1>;
            power_enable = <1>;
            lcd_55power: lcd-55power {
                   gpios = <&gpio1 4 GPIO_ACTIVE_HIGH>;
                   pinctrl-names = "default";
                   pinctrl-0 = <&lcd_panel_55power>;
                   rockchip,delay = <6>;
            };         
            lcd_rst: lcd-rst {
                   gpios = <&gpio4 29 GPIO_ACTIVE_HIGH>;
                   pinctrl-names = "default";
                   pinctrl-0 = <&lcd_panel_reset>;
                   rockchip,delay = <6>;
            };
        };
    ⋯
};

⋯

&pinctrl {

    lcd-panel {
        lcd_panel_reset: lcd-panel-reset {
            rockchip,pins = <4 29 RK_FUNC_GPIO &pcfg_pull_down>;
    };

    lcd_panel_enable: lcd-panel-enable {
        rockchip,pins = <1 24 RK_FUNC_GPIO &pcfg_pull_down>;
    };

    lcd_panel_55power: lcd-panel-55power {
        rockchip,pins = <1 4 RK_FUNC_GPIO &pcfg_pull_down>;
    };
};
```
Power control pin of LCD is defined here:
```
lcd_panel_reset：GPIO4_D5(GPIO_ACTIVE_HIGH)
lcd_panel_enable：GPIO1_D0(GPIO_ACTIVE_HIGH)
lcd_panel_55power：GPIO1_A4(GPIO_ACTIVE_HIGH)
```
All are high level effective, please refer to the section of **"GPIO"** for specific pin configuration.
####  Backlight Configure

A backlight interface is set outside the Core-3399-JD4 development board to control the backlight of the screen.

In the DTS file : `rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1.dts` configured in the backlight information, as follows:
```
&backlight {
    status = "okay";
    pwms = <&pwm1 0 50000 1>;
    brightness-levels = <
             /*0 23  23  24  24  25  25  26
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             87  87  87  87  87  87  87  87
             88  89  90  91  92  93  94  95*/
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
            192 193 194 195 196 197 198 198
            199 199 200 201 201 202 203 204
            205 206 207 208 209 210 211 212
            213 214 215 216 217 218 219 220
            221 222 223 224 225 226 227 228
            229 230 231 232 233 234 234 235
            236 237 238 239 240 241 242 243
            244 245 246 247 248 249 250 250
            250 250 250 250 250 250 250 250 >;
    default-brightness-level = <0>;
    enable-gpios = <&gpio1 24 GPIO_ACTIVE_HIGH>;
    //polarity = <1>;
    pinctrl-names = "default";
    pinctrl-0 = <&lcd_panel_enable>;
};
```



### Configure display timing


Different from EDP screen, *Timing* of MIPI screen is written in DTS file. The following statement can be seen in `rk3399-roc-pc-plus-mipi101-JDM101014_BC45_A1.dts`.
```
display-timings {
    native-mode = <&timing0>;
    timing0: timing0 {
    clock-frequency = <67000000>;//<80000000>
    hactive = <800>;//<768>;
    vactive = <1280>;
    hsync-len = <20>;   //20, 50
    hback-porch = <20>; //50, 56
    hfront-porch = <32>;//50, 30
    vsync-len = <4>;
    vback-porch = <4>;
    vfront-porch = <8>;
    hsync-active = <0>;
    vsync-active = <0>;
    de-active = <0>;
    pixelclk-active = <0>;
    };
};
```
### Brief Description of Driving Power on Sequence
#### Kernel
In `kernel/drivers/gpu/drm/panel/panel-simple.c` can be seen in `panel_simple_probe` initialization function to initialize the access sequence of functions.
```
static int panel_simple_probe(struct device *dev, const struct panel_desc *desc){
···
 panel->base.funcs = &panel_simple_funcs;
···
}
```

In `kernel/drivers/gpu/drm/panel/panel-simple.c`,you can see ：
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

Initialization mipi screen after the electricity needs to send instruction to make it work, can be in kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-mipi8.dts can see mipi initialization in the command list:
```
&dsi {      
    ⋯
      panel-init-sequence = [
        05 78 01 11
        05 32 01 29
    ];

    panel-exit-sequence = [
        05 00 01 28
        05 00 01 10
    ];
    ⋯
};
```
Please refer to the following attachment for the command format and instructions:
[Rockchip DRM Panel Porting Guide.pdf](http://www.t-firefly.com/ueditor/php/upload/file/20171213/1513128959299913.pdf)

Send instructions can be seen in the `kernel/drivers/gpu/drm/panel/panel-simple.c` :
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

#### U-boot
Send instructions can be seen in the `u-boot/drivers/video/rockchip-dw-mipi-dsi.c` :
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
## EDP_DTS configuration
### Pin Configuration
For pin configuration, refer to the File：`kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-edp.dts`
#### Chip Power on Configuration
with the file: `kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-edp.dts`, we can see from the file the following statement:

```
 edp_panel: edp-panel {
		compatible = "simple-panel";
		bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;

		backlight = <&backlight>;

		display-timings {
			native-mode = <&timing0>;

			timing0: timing0 {
				clock-frequency = <200000000>;
				hactive = <2560>;
				vactive = <1600>;
				hfront-porch = <48>;
				hsync-len = <80>;
				hback-porch = <32>;
				vfront-porch = <2>;
				vsync-len = <39>;
				vback-porch = <5>;
				hsync-active = <0>;
				vsync-active = <0>;
				de-active = <0>;
				pixelclk-active = <0>;
			};
		};

		ports {
			panel_in_edp: endpoint {
				remote-endpoint = <&edp_out_panel>;
			};
		};

		power_ctr: power_ctr {
               rockchip,debug = <1>;
               lcd_en: lcd-en {
                       gpios = <&gpio2 5 GPIO_ACTIVE_HIGH>;
					   pinctrl-names = "default";
					   pinctrl-0 = <&lcd_panel_enable>;
                       rockchip,delay = <1000>;
               };
			   /*
               lcd_edp_hpd: lcd-edp-hpd {
                       gpios = <&gpio1 20 GPIO_ACTIVE_HIGH>;
					   pinctrl-names = "default";
					   pinctrl-0 = <&lcd_edp_hpd_enable>;
                       rockchip,delay = <10>;
               };
			   */
       };
	};


···
&pinctrl {
  lcd-panel {
    lcd_panel_reset: lcd-panel-reset {
      rockchip,pins = <4 29 RK_FUNC_GPIO &pcfg_pull_up>;
    };
    lcd_panel_enable: lcd-panel-enable {
      rockchip,pins = <1 1 RK_FUNC_GPIO &pcfg_pull_up>;
    };
        lcd_panel_pwr_en: lcd-panel-pwr-en {
            rockchip,pins = <2 5 RK_FUNC_GPIO &pcfg_pull_up>;
        };
  };
};

```
Power control pin of LCD is defined here:

```
lcd_en：GPIO2_A4(GPIO_ACTIVE_HIGH)
lcd-edp-hpd：GPIO1_D4(GPIO_ACTIVE_HIGH)

```
All are high level effective, please refer to the section of **"GPIO"** for specific pin configuration.

### Backlight Configure 
```
	backlight: backlight {
			status = "okay";
			compatible = "pwm-backlight";
			pwms = <&pwm0 0 25000 0>;
			enable-gpios = <&gpio1 24 GPIO_ACTIVE_HIGH>;
			pinctrl-names = "default";
			pinctrl-0 = <&lcd_backlight_enable>;
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

### Brief Description of Driving Power on Sequence
#### Kernel

writing Timing in the *panel-simple.c*, directly to the short string matching in *drivers/gpu/drm/panel/panel-simple.c* file has the following statement

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

MODULE_DEVICE_TABLE(of, platform_of_match); The timing parameters are configured in the structure : *lg_lp079qx1_sp0v_mode*.
#### Uboot

The Timing is written in rockchip_panel.c, directly to the short string matching in *drivers/video/rockchip_panel.c* file has the following statement :
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
The timing parameters are configured in the structure:*lg_lp079qx1_sp0v_mode*.