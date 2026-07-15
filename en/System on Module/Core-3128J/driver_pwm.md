# PWM Use

Firefly-RK3128 development board has 4 PWMs, namely PWM0 ~ PWM3:

* PWM0/GPIO0_D2: exported in the extension board
* PWM1/GPIO0_D3: used as AUX_DET signal internally
* PWM2/GPIO0_D4: used as RTC_INT signal internally
* PWM3: used by infrared transceivers

This chapter mainly describes how to configure PWM. The PWM driver of Firefly-RK3128 is:
```
kernel/drivers/pwm/pwm-rockchip.c
```

## Configuration steps

Configuration of PWM mainly includes the following three steps: configure PWM DTS nodes, configure PWM kernel driver, and control PWM device.

## Create PWM DTS Node

The following PWM nodes are defined in kernel/arch/arm/boot/dts/rk312x.dtsi, as shown below:
```
pwm0: pwm@20050000 {
	compatible = "rockchip,rk-pwm";
	reg = <0x20050000 0x10>;
	#pwm-cells = <2>;
	pinctrl-names = "default";
	pinctrl-0 = <&pwm0_pin>;
	clocks = <&clk_gates7 10>;
	clock-names = "pclk_pwm";
	status = "disabled";
};

 pwm1: pwm@20050010 {
     compatible = "rockchip,rk-pwm";
     reg = <0x20050010 0x10>;
     #pwm-cells = <2>;
     pinctrl-names = "default";
     pinctrl-0 = <&pwm1_pin>;
     clocks = <&clk_gates7 10>;
     clock-names = "pclk_pwm";
     status = "disabled";
 };

 pwm2: pwm@20050020 {
      compatible = "rockchip,rk-pwm";
      reg = <0x20050020 0x10>;
      #pwm-cells = <2>;
      pinctrl-names = "default";
      pinctrl-0 = <&pwm2_pin>;
      clocks = <&clk_gates7 10>;
      clock-names = "pclk_pwm";
      status = "disabled";
 };

 pwm3: pwm@20050030 {
       compatible = "rockchip,rk-pwm";
       reg = <0x20050030 0x10>;
       #pwm-cells = <2>;
       pinctrl-names = "default";
       pinctrl-0 = <&pwm3_pin>;
       clocks = <&clk_gates7 10>;
       clock-names = "pclk_pwm";
       status = "disabled";
 };
```

To use PWM0, you need to add to file kernel/arch/arm/boot/dts/rk3128-fireprime.dts:
```
&pwm0 {
       status = "okay";
};
```

## Configure PWM kernel driver

The PWM driver is in file kernel/drivers/pwm/pwm-rockchip.c

### Control PWM Device

Now you can use the PWM node created above to control devices, as shown in the following steps:

#### (1). Include the following header file into the device driver files to be controlled by PWM:
```
#include <linux/pwm.h>
```
This header file mainly includes PWM function interface.

#### (2). Request for PWM    
You can use the pwm_request function to request the PWM device. For example:
```
struct pwm_device * pwm0=NULL;pwm0= pwm_request(0, “backlight-pwm”);
```
See The API of PWM device for more detail of this function.

#### (3). Configure PWM 
You can use the function pwm_config to set the PWM configuration. For example:
```
pwm_config(pwm0, 500000, 1000000)；
```
See The API of PWM device for more detail of this function.

#### (4). PWM enabling function
Then you start the pwm device using the pwm_enable function. For example：
```
int pwm_enable(struct pwm_device *pwm);
```
See The API of PWM device for more detail of this function.

### The API of PWM device
```
/**
 * pwm_request() - request a PWM device
 * @pwm_id: global PWM device index
 * @label: PWM device label
 *
 * This function is deprecated, use pwm_get() instead.
 */
 struct pwm_device *pwm_request(int pwm_id, const char *label);


/**
 * pwm_free() - free a PWM device
 * @pwm: PWM device
 *
 * This function is deprecated, use pwm_put() instead.
 */
 void pwm_free(struct pwm_device *pwm);


/**
 * pwm_config() - change a PWM device configuration
 * @pwm: PWM device
 * @duty_ns: "on" time (in nanoseconds)
 * @period_ns: duration (in nanoseconds) of one cycle
 */
 int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);


/**
 * pwm_enable() - start a PWM output toggling
 * @pwm: PWM device
 */
 int pwm_enable(struct pwm_device *pwm);


/**
 * pwm_disable() - stop a PWM output toggling
 * @pwm: PWM device
 */
 void pwm_disable(struct pwm_device *pwm);
 
```