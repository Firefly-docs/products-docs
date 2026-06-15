# LCD
## Introduction

ROC-3399-PC-PLUS development board supports two LCD screen interfaces by default, one is MIPI (compatible with 10.1 in. and 8 in. ), the other is EDP(10.1 in.). The positions of interfaces on the board are shown as follows:

10.1 in. MIPI
![](../../../rk3399_img/Core-3399-JD4/pc+_mipi101_1.jpg)

8 in. MIPI
![](../../../rk3399_img/Core-3399-JD4/panel_mipi8.jpg)

10.1 in. EDP
![](../../../rk3399_img/Core-3399-JD4/panel_edp101.jpg)


## MIPI_DTS configuration
### Pin configuration


ROC-3399-PC-PLUS  DTS file path : 

8 in. screen : kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-mipi8.dts


From the file `rk3399-roc-pc-plus-mipi8.dts` we can see the following statement:

```
...

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
  
...
```
Power control pin of LCD is defined here:
```
lcd_55power:(GPIO1_A3)GPIO_ACTIVE_HIGH
lcd_rst:(GPIO4_D5)GPIO_ACTIVE_HIGH
lcd_panel_enable：(GPIO1_D0)GPIO_ACTIVE_HIGH
```
All are high level effective, please refer to the section of **"GPIO"** for specific pin configuration.
### MIPI Configure backlight

A backlight interface is set outside the ROC-3399-PC-PLUS development board to control the backlight of the screen.

In the DTS file : *kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-mipi8.dts* configured in the backlight information, as follows:
```
...

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
    enable-gpios = <&gpio1 24 GPIO_ACTIVE_HIGH>;
    pinctrl-names = "default";
    pinctrl-0 = <&lcd_panel_enable>;
};

...
};
```



### Configure display timing


Different from EDP screen, *Timing* of MIPI screen is written in DTS file. The following statement can be seen in *kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-mipi8.dts*.
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
* Kernel
In kernel/drivers/gpu/drm/panel/panel-simple.c can be seen in `panel_simple_probe` initialization function to initialize the access sequence of functions.
```
static int panel_simple_probe(struct device *dev, const struct panel_desc *desc){
···
 panel->base.funcs = &panel_simple_funcs;
···
}
```

In kernel/drivers/gpu/drm/panel/panel-simple.c,you can see ：
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
            status = "okay";      
        ...
         panel-init-sequence = [
            15 00 02 E0 00
            15 00 02 E1 93
            15 00 02 E2 65
            15 00 02 E3 F8
            15 00 02 80 03

            15 00 02 E0 04
            15 00 02 2D 03
            15 00 02 09 14

            15 00 02 E0 01
            15 00 02 4B 04

            15 00 02 17 00
            15 00 02 18 B1
            15 00 02 19 01
            15 00 02 1A 00
            15 00 02 1B B1
            15 00 02 1C 01

            15 00 02 1F 3E
            15 00 02 20 2D
            15 00 02 21 2D
            15 00 02 22 0E
        15 00 02 35 0F

            15 00 02 37 19

            15 00 02 38 05
            15 00 02 39 00
            15 00 02 3A 01
            15 00 02 3C 78
        15 00 02 3D FF
            15 00 02 3E FF
            15 00 02 3F FF

            15 00 02 40 06
            15 00 02 41 A0
        15 00 02 4B 00

            15 00 02 55 0F
            15 00 02 56 01
            15 00 02 57 69
            15 00 02 58 0A
            15 00 02 59 0A
            15 00 02 5A 28
            15 00 02 5B 19
            //GAMMA
            15 00 02 5D 7F
            15 00 02 5E 72
            15 00 02 5F 67
            15 00 02 60 5F
            15 00 02 61 5F
            15 00 02 62 51
            15 00 02 63 58
            15 00 02 64 44
            15 00 02 65 5C
            15 00 02 66 5A
            15 00 02 67 58
            15 00 02 68 74
            15 00 02 69 5D
            15 00 02 6A 5F
            15 00 02 6B 4D
            15 00 02 6C 45
            15 00 02 6D 36
            15 00 02 6E 24
            15 00 02 6F 0A
            15 00 02 70 7F
            15 00 02 71 72
            15 00 02 72 67
            15 00 02 73 5F
            15 00 02 74 5F
            15 00 02 75 51
            15 00 02 76 58
            15 00 02 77 44
            15 00 02 78 5C
            15 00 02 79 5A
            15 00 02 7A 58
            15 00 02 7B 74
            15 00 02 7C 5D
            15 00 02 7D 5F
            15 00 02 7E 4D
            15 00 02 7F 4D
            15 00 02 80 36
            15 00 02 81 24
            15 00 02 82 0A

            15 00 02 E0 02

            15 00 02 00 47
            15 00 02 01 47
            15 00 02 02 45
            15 00 02 03 45
            15 00 02 04 4B
            15 00 02 05 4B
            15 00 02 06 49
            15 00 02 07 49
            15 00 02 08 41
            15 00 02 09 1F
            15 00 02 0A 1F
            15 00 02 0B 1F
            15 00 02 0C 1F
            15 00 02 0D 1F
            15 00 02 0E 1F
            15 00 02 0F 43
            15 00 02 10 1F
            15 00 02 11 1F
            15 00 02 12 1F
            15 00 02 13 1F
            15 00 02 14 1F
            15 00 02 15 1F

            15 00 02 16 46
            15 00 02 17 46
            15 00 02 18 44
            15 00 02 19 44
            15 00 02 1A 4A
            15 00 02 1B 4A
            15 00 02 1C 48
            15 00 02 1D 48
            15 00 02 1E 40
            15 00 02 1F 1F
            15 00 02 20 1F
            15 00 02 21 1F
            15 00 02 22 1F
            15 00 02 23 1F
            15 00 02 24 1F
            15 00 02 25 42
            15 00 02 26 1F
            15 00 02 27 1F
            15 00 02 28 1F
            15 00 02 29 1F
            15 00 02 2A 1F
            15 00 02 2B 1F

            15 00 02 2C 11
            15 00 02 2D 0F
            15 00 02 2E 0D
            15 00 02 2F 0B

            15 00 02 30 09
            15 00 02 31 07
            15 00 02 32 05
            15 00 02 33 18
            15 00 02 34 17
            15 00 02 35 1F
            15 00 02 36 01
            15 00 02 37 1F
            15 00 02 38 1F
            15 00 02 39 1F
            15 00 02 3A 1F
            15 00 02 3B 1F
            15 00 02 3C 1F
            15 00 02 3D 1F
            15 00 02 3E 1F
            15 00 02 3F 13
            15 00 02 40 1F
            15 00 02 41 1F

            15 00 02 42 10
            15 00 02 43 0E
            15 00 02 44 0C
            15 00 02 45 0A
        15 00 02 46 08
            15 00 02 47 06
            15 00 02 48 04
            15 00 02 49 18
            15 00 02 4A 17
            15 00 02 4B 1F
        15 00 02 4C 00
            15 00 02 4D 1F
        15 00 02 4E 1F
            15 00 02 4F 1F
            15 00 02 50 1F
            15 00 02 51 1F
            15 00 02 52 1F
            15 00 02 53 1F
            15 00 02 54 1F
            15 00 02 55 12
            15 00 02 56 1F
        15 00 02 57 1F

            15 00 02 58 40
        15 00 02 59 00
        15 00 02 5A 00
            15 00 02 5B 30
            15 00 02 5C 03
            15 00 02 5D 30
            15 00 02 5E 01
            15 00 02 5F 02
            15 00 02 60 03
            15 00 02 61 01
            15 00 02 62 02
            15 00 02 63 03
            15 00 02 64 6B
            15 00 02 65 00
            15 00 02 66 00
            15 00 02 67 73
            15 00 02 68 05
            15 00 02 69 06
            15 00 02 6A 6B
            15 00 02 6B 08
            15 00 02 6C 00
            15 00 02 6D 05
            15 00 02 6E 05
            15 00 02 6F 88
            15 00 02 70 00
            15 00 02 71 00
            15 00 02 72 06
            15 00 02 73 7B
            15 00 02 74 00
            15 00 02 75 07
            15 00 02 76 00
            15 00 02 77 5D
            15 00 02 78 17
            15 00 02 79 1F
            15 00 02 7A 00
            15 00 02 7B 00
            15 00 02 7C 00
            15 00 02 7D 03
            15 00 02 7E 7B

            15 00 02 E0 01
            15 00 02 0E 00

            15 00 02 E0 03
            15 00 02 98 3F

            15 00 02 E0 04
            15 00 02 09 10
            15 00 02 0E 48
            15 00 02 2B 2B
            15 00 02 2E 44

            15 00 02 E0 00
            15 00 02 E6 02
            15 00 02 E7 02
        05 78 01 11
        05 05 01 29
        15 00 02 35 00 
        ];

        panel-exit-sequence = [
            05 05 01 28
            05 78 01 10
        ];
        ...
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

* U-boot
Send instructions can be seen in the u-boot/drivers/video/rockchip-dw-mipi-dsi.c :
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
### Pin configuration

RK3399-ROC-PC-PLUS's SDK with EDP DSI DTS file: *kernel/arch/arm64/boot/dts/rockchip/rk3399-roc-pc-plus-edp.dts*, we can see from the file the following statement:

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
lcd_en:(GPIO2_A4)GPIO_ACTIVE_HIGH
lcd-edp-hpd:(GPIO1_D4)GPIO_ACTIVE_HIGH

```
All are high level effective, please refer to the section of **"GPIO"** for specific pin configuration.

### EDP Configure backlight
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

### EDP configuration displays timing
*  **kernel**

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
* **U-boot**

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