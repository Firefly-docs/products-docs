# IR Use

## IR remote control configuration

The AIO-3288J development board can be connected to the infrared transceiver sensor IR to realize the remote control function.


The AIO-3288J IR wiring position is shown in the red box below:

![](../../../rk3288_img/AIO-3288J/ir_interface.png)


This article describes how to use and configure the IR to work properly. You can do this in two parts:

* Modify the kernel driver: modify in kernel space, which is applicable to both Linux and Android OS.
* Modify the key mapping: modify in user space (limited to Android OS).

## Kernel driver

In Linux kernel, IR driver only supports NEC coding format. The text below describes the method to configure infrared remote control in kernel.

The involved files: `drivers/input/remotectl/rockchip_pwm_remotectl.c`

### Define the relevant data structure

The text below describes the steps to define data structure:

* Add key table structure array:

```c
static struct rkxx_remote_key_table remote_key_table_r66[12] = {
    {0xeb, KEY_POWER},        // Power
    //Control
    {0xa3, 250},              // Settings
    {0xec, KEY_MENU},         // Menu
    {0xfc, KEY_UP},           // Up
    {0xfd, KEY_DOWN},         // Down
    {0xf1, KEY_LEFT},         // Left
    {0xe5, KEY_RIGHT},        // Right
    {0xf8, KEY_REPLY},        // Ok
    {0xb7, KEY_HOME},         // Home
    {0xfe, KEY_BACK},         // Back
    // Vol
    {0xa7, KEY_VOLUMEDOWN},   // Vol-
    {0xf4, KEY_VOLUMEUP},     // Vol+
};
```

Note:The first column represents key, and the second column represents the key code to be responded.

* Add key structure array

```c
static struct rkxx_remotectl_button remotectl_button[] =
{
    //...
    {
        .usercode = 0xff00,/* need to get the usercode in next step */
        .nbuttons =  12,/* number of buttons */  
        .key_table = &remote_key_table_r66[0],/* key table */
    },
    // ...
};
```

* Note:
    + usercode represents user code, and each IR has a corresponding user code;
    + nbuttons represents number of remote control buttons;
    + key_table represents the address of key table structure array added in the first step.

#### How to obtain user code and IR key

Obtain user code and key in remotectl_do_something function:

```c
case RMC_USERCODE:
{
    //ddata->scanData <<= 1;
    //ddata->count ++;
    if ((RK_PWM_TIME_BIT1_MIN < ddata->period) && (ddata->period < RK_PWM_TIME_BIT1_MAX)){
        ddata->scanData |= (0x01<<ddata->count);;
    }
    ddata->count ++;
    if (ddata->count == 0x10){//16 bit user code
        DBG_CODE("GET USERCODE=0x%x\n",((ddata->scanData) & 0xffff));
        if (remotectl_keybdNum_lookup(ddata)){
            ddata->state = RMC_GETDATA;
            ddata->scanData = 0;
            ddata->count = 0;
        }else{                //user code error
            ddata->state = RMC_PRELOAD;
        }
    }
}
```

Note: the user can use DBG_CODE() function to print user code.

Add user code and key to remotectl_button array:

```c
case RMC_GETDATA:
{
    //ddata->count ++;
    //ddata->scanData <<= 1;
    #ifdef CONFIG_FIREFLY_POWER_LED
    mod_timer(&timer_led,jiffies + msecs_to_jiffies(50));
    remotectl_led_ctrl(0);
    #endif
    if ((RK_PWM_TIME_BIT1_MIN < ddata->period) && (ddata->period < RK_PWM_TIME_BIT1_MAX)){
        ddata->scanData |= (0x01<<ddata->count);
    }
    ddata->count ++;
    if (ddata->count == 0x10){
        DBG_CODE("RMC_GETDATA=%x\n",(ddata->scanData>>8));
        if ((ddata->scanData&0x0ff) == ((~ddata->scanData >> 8)&0x0ff)){
            if (remotectl_keycode_lookup(ddata)){
                ddata->press = 1;
                ...
            }
            ...
        }
        ...
    }
}
```

Note: the user can use DBG_CODE()function to print key.

#### Compile IR driver into kernel

The steps to compile IR driver into kernel are as follows:

(1). Add the following configuration into the configuration file drivers/input/remotectl/Kconfig:

```c
config RK_REMOTECTL_PWM
    bool "rkxx remoctrl pwm0 capture"
default n
```

(2). Modify Makefile under drivers/input/remotectl, and add the following compiling option:

```makefile
obj-$(RK_REMOTECTL_PWM)      += rk_pwm_remotectl.o
```

(3). Use make menuconfig under kernel path, and check IR driver according to the following method.

```c
Device Drivers
  --->Input device support
  ----->  [*]   rkxx remotectl
  ------->[*]   rkxx remoctrl pwm0 capture.
```

Save, execute make command to compile this driver into kernel.

#### Android key mapping

File /system/usr/keylayout/ff680000_pwm.kl is used for mapping the key mapping obtained at Linux layer to the corresponding key on Android. The user can add or modify the contents of this file to realize different key mapping.

The contents of this file are shown below:

```c
key 28    ENTER
key 116   POWER             WAKE
key 158   BACK
key 139   MENU
key 217   SEARCH
key 232   DPAD_CENTER
key 108   DPAD_DOWN
key 103   DPAD_UP
key 102   HOME
key 105   DPAD_LEFT
key 106   DPAD_RIGHT
key 115   VOLUME_UP
key 114   VOLUME_DOWN
key 143   NOTIFICATION      WAKE
key 113   VOLUME_MUTE
key 388   TV_KEYMOUSE_MODE_SWITCH
key 400   TV_MEDIA_MULT_BACKWARD
key 401   TV_MEDIA_MULT_FORWARD
key 402   TV_MEDIA_PLAY_PAUSE
key 64    TV_MEDIA_PLAY
key 65    TV_MEDIA_PAUSE
key 66    TV_MEDIA_STOP
```

Note: modify this file through adb, which will enter into force after reboot.