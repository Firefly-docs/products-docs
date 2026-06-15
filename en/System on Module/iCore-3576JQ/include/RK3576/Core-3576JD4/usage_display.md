# Display Usage

RK3576 has 3 vp (Video Port), every vp support some display interfaces. Check vop_out node in device tree rk3576.dtsi for details.

For example: vp0 supports dsi, hdmi/edp, dp0~2 display.

Max resolution@fps for each vp:
* vp0 Up to 4K@120Hz
* vp1 Up to 2560x1600@60Hz
* vp2 Up to 1920x1080@60Hz

If you bind HDMI0 to vp0, and your hardware supports HDMI2.1, then HDMI0 can output 4K120Hz.

Bind a screen to every vp then you can get three screens display at same time, with same or different contents.

## AIO-3576JQ Display Configuration

Core-3576JD4 supports different type of displays, but AIO-3576JQ only export HDMI

If you need other display interfaces while designing your own motherboard, you can refer to our other products' dts, like:
```
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc-mipi101-M101014-BE45-A1.dts
```