# PWM Use

## Introduction

Firefly-RK3288 development board has 4-way PWM outputs, which are PWM0 ~ PWM3, this chapter mainly describes how to configure PWM.

The PWM driver file:

```c
kernel/drivers/pwm/pwm-rockchip.c
```

## Data structure

### pwm_device structure

```c
struct pwm_device {
    const char      *label;
    unsigned long       flags;
    unsigned int        hwpwm;
    unsigned int        pwm;//pwm channel
    struct pwm_chip     *chip;
    void            *chip_data;
    unsigned int        period; /* in nanoseconds */
};
```

### pwm_chip structure

This structure is a abstract PWM controller.

```c
struct pwm_chip {
    struct device       *dev; //provide PWM device
    struct list_head    list; //internally used node list
    const struct pwm_ops    *ops; //callback function of this PWM controller
    int  base; //number of the first PWM controlled by this device
    unsigned int        npwm; //number of PWMs controlled by this device
    struct pwm_device   *pwms;
    struct pwm_device * (*of_xlate)(struct pwm_chip *pc,const struct of_phandle_args *args);
    unsigned int of_pwm_n_cells;
    bool can_sleep;
};
```

## Configuration steps

Configuration of PWM mainly includes the following three steps: configure PWM DTS nodes, configure PWM kernel driver, and control PWM device.

### Configure PWM DTS node

The following PWM nodes are defined in `kernel/arch/arm/boot/dts/rk3288.dtsi`, as shown below:

```dts
pwm1: pwm@ff680010 {
      compatible = "rockchip,rk-pwm";
      reg = ;
      #pwm-cells = ;
      pinctrl-names = "default";
      pinctrl-0 = ;
      clocks = ;
      clock-names = "pclk_pwm";
      status = "okay";
};
```

Note: ff680010 is the address of PWM1 register.

In order to use pwm1, add it into `kernel/arch/arm/boot/dts/firefly-rk3288.dts`:

```dts
&pwm1 {
    status = "okay";
};
```

## Configure PWM kernel driver

PWM driver is located on the file `kernel/drivers/pwm/pwm-rockchip.c`.

Modify the following codes of this file:

```c
static const struct of_device_id rk_pwm_of_match[] = {
    { .compatible = "rockchip,pwm",       .data = &rk_pwm_data_v1,},
    { .compatible =  "rockchip,rk-pwm",   .data = &rk_pwm_data_v2,},
    { .compatible =  "rockchip,vop-pwm",  .data = &rk_pwm_data_v3,},
    { }
};
```

Add the compatible = "rockchip,rk-pwm" in the previous step to the above code.

## Control PWM device

The user can use the PWM node generated from the above steps in the other driver file. The specific methods are as follows:

### (1) Include the following header file into the device driver files to be controlled by PWM

```c
#include <linux/pwm.h>
```

This header file mainly includes PWM function interface.

### (2) Request for PWM

```c
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

Function requests for PWM. For example:

```c
struct pwm_device * pwm0 = NULL;
pwm0 = pwm_request(0, “backlight-pwm”);
```

Parameter pwm_id represents the channel for which PWM is to be requested, label is the label taken by this PWM.

### (3) Configure PWM

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

Configure duty ratio of PWM, for example:

```c
pwm_config(pwm0, 500000, 1000000);
```

Parameter pwm represents the pwm_device requested in the previous step. duty_ns represents the activating duration of duty ratio, and its unit is ns. period_ns represents the period of PWM, and its unit is ns.

### (4) PWM enabling function

```c
int pwm_enable(struct pwm_device *pwm);
```

It is used for enabling PWM, for example:

```c
pwm_enable(pwm0);
```

Parameter pwm represents the pwm_device to be enabled.

Controlling PWM output mainly uses the following interface functions:

```c
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

* Function: for requesting pwm
* Parameter:
    + pwm_id: pwm channel to be requested.
    + label: label taken by the pwm requested.

```c
void pwm_free(struct pwm_device *pwm);
```

* Function: for releasing the requested pwm
* Parameter:
    + pwm: pwm structure to be released

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

* Function: for configuring the duty ratio of pwm
* Parameter:
    + pwm: pwm to be configured
    + duty_ns: activated duration of pwm duty ratio, the unit is ns
    + period_ns: period of pwm duty ratio, the unit is ns

```c
int pwm_enable(struct pwm_device *pwm);
```

* Function: enable pwm
* Parameter:
    + pwm: pwm to be enabled

```c
void pwm_disable(struct pwm_device *pwm); 
```

* Function: forbid pwm
* Parameter:
    + pwm: pwm to be forbidden