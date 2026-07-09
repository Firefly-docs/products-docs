# LCD

## Introduction

Face-RK3399 supports MIPI screen, The interface corresponds to the position of the board as shown in the figure below:

![](img/mipi.png)

### Configuration

* DTS

After adding MIPI hardware interface, user should configure the `display-timings` of MIPI screen in `kernel/arch/arm64/boot/dts/rockchip/firefly-face-mipi8.dts`:

```
display-timings {
		native-mode = <&timing0>;
		timing0: timing0 {
			clock-frequency = <67000000>;//<80000000>;
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
};
```

* Kernel

In `kernel/drivers/gpu/drm/panel/panel-simple.c`, can see function `panel_simple_probe` initializes the funciton of get time sequence.

```
static int panel_simple_probe(struct device *dev, const struct panel_desc *desc){
···
 panel->base.funcs = &panel_simple_funcs;
···
}
```

`panel_simple_get_timings` is defined in `kernel/drivers/gpu/drm/panel/panel-simple.c`:

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

Mipi screen needs to send initialization command to work after power up. You can see the initialization command list in `/kernel/arch/arm64/boot/dts/rockchip/firefly-face-mipi8.dts`:

```
&mipi_dsi {    
            status = "okay";      
        ...
            panel-init-sequence = [                
                05 20 01 29                
                05 96 01 11 
                ...           
            ];            
            
            panel-exit-sequence = [                
                05 05 01 28                
                05 78 01 10            
            ];
        ...
};
```

The format of the cammands can refer to the following file: [Rockchip DRM Panel Porting Guide.pdf](http://www.t-firefly.com/ueditor/php/upload/file/20171213/1513128959299913.pdf)

The commands are sent in `kernel/drivers/gpu/drm/panel/panel-simple.c`

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

* U-Boot

The commands are sent in `u-boot/drivers/video/rockchip-dw-mipi-dsi.c`

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

### EDP lcd
Face-rk3399 V2.0 support EDP display output refer to [interface definition](http://wiki.t-firefly.com/en/Face-RK3399/interface_definition.html)

To config Timing dts file，kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-face-edp15.dts：

```
display-timings {
        native-mode = <&timing0>;
        timing0: timing0 {
                clock-frequency = <140000000>;
                hactive = <1920>;
                vactive = <1080>;
                hfront-porch = <100>;
                hsync-len = <40>;
                hback-porch = <40>;
                vfront-porch = <10>;
                vsync-len = <10>;
                vback-porch = <10>;
                hsync-active = <0>;
                vsync-active = <0>;
                de-active = <0>;
                pixelclk-active = <0>;
        };                                                                                                                             
};
```

The power control gpio：

lcd_en:gpios = <&gpio3 26 GPIO_ACTIVE_HIGH>;