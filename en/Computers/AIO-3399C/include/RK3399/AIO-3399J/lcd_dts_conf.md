```
/ {
    model = "AIO Board lvds (Android)";
    compatible = "rockchip,android", "rockchip,rk3399-firefly-lvds", "rockchip,rk3399";

    test-power {
        status = "okay";
    };
    ...
    &dsi {
    status = "okay";
    dsi_panel: panel {
        compatible ="simple-panel-dsi";
        reg = <0>;
        //ddc-i2c-bu
        //power-supply = <&vcc_lcd>;
        //pinctrl-0 = <&lcd_panel_reset &lcd_panel_enable>;
        backlight = <&backlight>;
        /*
        enable-gpios = <&gpio1 1 GPIO_ACTIVE_LOW>;
        reset-gpios = <&gpio4 29 GPIO_ACTIVE_LOW>;
        */
        dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;
        dsi,format = <MIPI_DSI_FMT_RGB888>;
        //bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;
        dsi,lanes = <4>;

        dsi,channel = <0>;

        enable-delay-ms = <35>;
        prepare-delay-ms = <6>;

        unprepare-delay-ms = <0>;
        disable-delay-ms = <20>;

        size,width = <120>;
        size,height = <170>;

        status = "okay";
        ...
        power_ctr: power_ctr {
               rockchip,debug = <0>;
               lcd_pwr_en: lcd-pwr-en {
                       gpios = <&gpio3 16 GPIO_ACTIVE_HIGH>;
                       pinctrl-names = "default";
                       pinctrl-0 = <&lcd_panel_pwr_en>;
                       rockchip,delay = <10>;
               };

               lcd_rst: lcd-rst {
                       gpios = <&gpio2 27 GPIO_ACTIVE_HIGH>;
                       pinctrl-names = "default";
                       pinctrl-0 = <&lcd_panel_reset>;
                       rockchip,delay = <6>;
               };
           };
      ...
      &pinctrl {
      lcd-panel {
        lcd_panel_reset: lcd-panel-reset {
            rockchip,pins = <2 27 RK_FUNC_GPIO &pcfg_pull_down>;
        };

        lcd_panel_pwr_en: lcd-panel-pwr-en {
            rockchip,pins = <3 16 RK_FUNC_GPIO &pcfg_pull_down>;
        };

    };
};
```

Power control pin of LCD is defined here:

```
lcd_pwr_en:(GPIO3_C0)GPIO_ACTIVE_HIGH
lcd_rst:(GPIO2_D3)GPIO_ACTIVE_HIGH
```