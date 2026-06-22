# Display module

## [10.1 inch LVDS display module](https://www.firefly.store/products)

### Product parameters

* Brand: Firefly
* Model: HSX101H40C
* Size: 10.1 inch
* Resolution: 1280x800
* Interface: LVDS
* Visual angle: 170°
* Touch screen: Multi-touch capacitive

### Kernel compilation

```bash
make firefly_defconfig
make -j8 firefly-rk3288-aio-3288j_lvds.img
```

### Reference materials

[Display module Datasheet](https://drive.google.com/open?id=1O0AyalT3W-2L1woR3Ui4EJFPWq-zDqAG)

[Adapter board schematic](https://drive.google.com/open?id=13CulGfOEJURq2GWqg0ETurQ6aQJKgvrb)

### Reference firmware

[AIO-3288J_LVDS](http://en.t-firefly.com/doc/download/page/id/45.html)

### Wiring method

![](../../../rk3288_img/AIO-3288J/module_display.png)

* Note: The yellow socket is TP socket while the red socket is backlit control socket.