# IR 使用

## 红外遥控配置

AIO-3288J开发板上可以接红外收发传感器 IR 实现遥控功能。


AIO-3288J的 IR 接线位置如下图红框所示：

![](../../../rk3288_img/AIO-3288J/ir_interface.png)


本文主要描述在开发板上如何配置红外遥控器。其配置步骤可分为两个部分：

1. 修改内核驱动：内核空间修改，Linux 和 Android 都要修改这部分的内容。
2. 修改键值映射：用户空间修改（仅限 Android 系统）。

## 内核驱动

在 Linux 内核中，IR 驱动仅支持 NEC 编码格式。以下是在内核中配置红外遥控的方法。

所涉及到的文件：`drivers/input/remotectl/rockchip_pwm_remotectl.c`

### 定义相关数据结构

**以下是定义数据结构的步骤：**  

* 添加键值表结构体数组

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

注：第一列为键值，第二列为要响应的按键码。

* 添加按键结构体数组

```c
static struct rkxx_remotectl_button remotectl_button[] =
{
    //...
    {   .usercode = 0xff00,/* need to get the usercode in next step */
        .nbuttons =  12,/* number of buttons */  
        .key_table = &remote_key_table_r66[0],/* key table */
    },
    // ...
};
```

注：

1. usercode 是用户码，每个IR都有一个对应的用户码；
2. nbuttons 是遥控按键个数；
3. key_table 是在第一步中添加的键值表结构体数组的地址。

### 如何获取用户码和IR 键值

在 remotectl_do_something 函数中获取用户码和键值：

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

注：用户可以使用 DBG_CODE() 函数打印用户码。

向 remotectl_button 数组添加用户码和键值：

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
                ...                     }
             ...                  }
          ...              }
}
```

注：用户可以使用 DBG_CODE() 函数打印键值。

### 将 IR 驱动编译进内核

将 IR 驱动编译进内核的步骤如下所示：

1.向配置文件 drivers/input/remotectl/Kconfig 中添加如下配置：

```c
config RK_REMOTECTL_PWM
    bool "rkxx remoctrl pwm0 capture"
default n
```

2.修改 drivers/input/remotectl 路径下的 Makefile,添加如下编译选项：

```makefile
obj-$(RK_REMOTECTL_PWM)      += rk_pwm_remotectl.o
```

3.在 kernel 路径下使用 make menuconfig ，按照如下方法将IR驱动选中。

```c
Device Drivers
  --->Input device support
  ----->  [*]   rkxx remotectl
  ------->[*]   rkxx remoctrl pwm0 capture.
```

保存后，执行 make 命令即可将该驱动编进内核。

## Android 键值映射

文件 `/system/usr/keylayout/ff680000_pwm.kl` 用于将 Linux 层获取的键值映射到 Android 上对应的键值。用户可以添加或者修改该文件的内容以实现不同的键值映射。

该文件内容如下所示：

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

注：通过 adb 修改该文件重启后即可生效。