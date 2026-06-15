# Display 使用

RK3576 拥有 3 路 vp (Video Port)，每一个 vp 端口都绑定了固定的显示控制器。详情可以查看设备树文件 rk3576.dtsi 中的 vop_out 节点。

比如 vp0 支持 dsi, hdmi/edp, dp0~2 的输出。

每一个 vp 都有各自所能输出的最大分辨率：
* vp0 最大可以输出 4K@120Hz
* vp1 最大可以输出 2560x1600@60Hz
* vp2 最大可以输出 1920x1080@60Hz

软件上如果把 HDMI0 显示控制器连接在 vp0 上，且硬件 phy 支持 HDMI2.1，则 HDMI0 支持 4K@120Hz 的输出。

如果给每一个 vpx 都单独分配一个显示控制器的话，便可支持同一时间的三屏同显（异显）。

## AIO-3576C 显示接口的配置

核心板虽然支持多种视频接口，但 AIO-3576C 只有 HDMI 接口。

客户在自行设计底板时候如果要用到其他接口，可以参考其他板型的 dts 比如：
```
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi
kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc-mipi101-M101014-BE45-A1.dts
```