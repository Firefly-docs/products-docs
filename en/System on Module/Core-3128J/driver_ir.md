# IR Use

## IR Configuration
The Firefly-RK3128 development board comes with IR sensor (between the USB OTG and headset jack) . This article describes how to use and configure the IR to work properly.  
You can do this in two parts:

* Modify the IR kernel driver. Works on both Linux and Android, which is low level hacking.
* Modify the key mapping if you are using Android, which is user space hacking.

## Kernel Driver

The IR driver only supports NEC encode format. Below is the instructions of how to add your IR remote to send the right key code in Linux kernel.  
Files involved:  

* DTS: kernel/arch/arm/boot/dts/rk3128-fireprime.dts
* Driver: kernel/drivers/input/remotectl/rk_pwm_remotectl.c

### IR Key tables
```
&remotectl {
	handle_cpu_id = <1>;
	ir_key1{
    	rockchip,usercode = <0xff00>;
        rockchip,key_table = <0xeb   KEY_POWER>,   
        					<0xa3   250>,   
                            <0xec   KEY_MENU>,   
                            <0xfc   KEY_UP>,   
                            <0xfd   KEY_DOWN>,   
                            <0xf1   KEY_LEFT>,   
                            <0xe5   KEY_RIGHT>,   
                            <0xf8   KEY_REPLY>,  
                            <0xb7   KEY_HOME>,   
                            <0xfe   KEY_BACK>,  
                            <0xa7   KEY_VOLUMEDOWN>,   
                            <0xf4   KEY_VOLUMEUP>;
    };
    
	ir_key2{
		rockchip,usercode = <0xff00>;
		rockchip,key_table = <0xf9	KEY_HOME>,
        					<0xbf	KEY_BACK>,
                            <0xfb	KEY_MENU>,
                            <0xaa	KEY_REPLY>,
                            <0xb9	KEY_UP>,
                            <0xe9	KEY_DOWN>,
                            <0xb8	KEY_LEFT>,
                            <0xea	KEY_RIGHT>,
                            <0xeb	KEY_VOLUMEDOWN>,
                            <0xef	KEY_VOLUMEUP>,
                            <0xf7	KEY_MUTE>,
                            <0xe7	KEY_POWER>,
                            <0xfc	KEY_POWER>,
                            <0xa9	KEY_VOLUMEDOWN>,
                            <0xa8	KEY_VOLUMEDOWN>,
                            <0xe0	KEY_VOLUMEDOWN>,
                            <0xa5	KEY_VOLUMEDOWN>,
                            <0xab    183>,
                            <0xb7    388>,
                            <0xf8    184>,
                            <0xaf    185>,
                            <0xed	KEY_VOLUMEDOWN>,
                            <0xee    186>,
                            <0xb3	KEY_VOLUMEDOWN>,
                            <0xf1	KEY_VOLUMEDOWN>,
                            <0xf2	KEY_VOLUMEDOWN>,
                            <0xf3	KEY_SEARCH>,
                            <0xb4	KEY_VOLUMEDOWN>,
                            <0xbe	KEY_SEARCH>;
    };
    
	ir_key3{
		rockchip,usercode = <0x1dcc>;
		rockchip,key_table =<0xee	KEY_REPLY>,
        					<0xf0	KEY_BACK>,
                            <0xf8	KEY_UP>,
                            <0xbb	KEY_DOWN>,
                            <0xef	KEY_LEFT>,
                            <0xed	KEY_RIGHT>,
                            <0xfc	KEY_HOME>,
                            <0xf1	KEY_VOLUMEUP>,
                            <0xfd	KEY_VOLUMEDOWN>,
                            <0xb7	KEY_SEARCH>,
                            <0xff	KEY_POWER>,
                            <0xf3	KEY_MUTE>,
                            <0xbf	KEY_MENU>,
                            <0xf9    0x191>,
                            <0xf5    0x192>,
                            <0xb3    388>,
                            <0xbe	KEY_1>,
                            <0xba	KEY_2>,
                            <0xb2	KEY_3>,
                            <0xbd	KEY_4>,
                            <0xf9	KEY_5>,
                            <0xb1	KEY_6>,
                            <0xfc	KEY_7>,
                            <0xf8	KEY_8>,
                            <0xb0	KEY_9>,
                            <0xb6	KEY_0>,
                            <0xb5	KEY_BACKSPACE>;
    };
};
```
Note:  

* usercode: every IR has a unique user code.  
* key_table: the mapping of scan code to kernel key code.

### Get User Code and Key Value

You can get the answer in the remotectl_do_something func:
```
case RMC_USERCODE: {
	if ((RK_PWM_TIME_BIT1_MIN < ddata->period) &&(ddata->period < RK_PWM_TIME_BIT1_MAX))
		ddata->scandata |= (0x01 << ddata->count);
	ddata->count++;
    if (ddata->count == 0x10) {
			DBG_CODE("USERCODE=0x%x\n", ddata->scandata);
            if (remotectl_keybd_num_lookup(ddata)) {
				ddata->state = RMC_GETDATA;
				ddata->scandata = 0;
				ddata->count = 0;
            } 
            else {if (rk_remote_print_code){
            	ddata->state = RMC_GETDATA;
				ddata->scandata = 0;
				ddata->count = 0;} else
				ddata->state = RMC_PRELOAD;
             }
    }
}
```
You can get the user code through DBG_CODE function.  
Add them to the remotectl_button array.
```
case RMC_GETDATA: {
	#ifdef CONFIG_FIREFLY_POWER_LEDmod_timer(&timer_led,jiffies + 											msecs_to_jiffies(50));remotectl_led_ctrl(0);
    #endif 
    if(!get_state_remotectl() && (ddata->keycode != KEY_POWER)) {
    	ledtrig_ir_activity();
    }
    if ((RK_PWM_TIME_BIT1_MIN < ddata->period) &&(ddata->period < RK_PWM_TIME_BIT1_MAX))
		ddata->scandata |= (0x01<<ddata->count);
	ddata->count++;
    if (ddata->count < 0x10) return;
	DBG_CODE("RMC_GETDATA=%x\n", (ddata->scandata>>8));
    if ((ddata->scandata&0x0ff) ==((~ddata->scandata >> 8) & 0x0ff)) {
    	if (remotectl_keycode_lookup(ddata)) {
				ddata->press = 1;
				input_event(ddata->input, EV_KEY,ddata->keycode, 1);
				input_sync(ddata->input);
				ddata->state = RMC_SEQUENCE;
        } 
        else {
			ddata->state = RMC_PRELOAD;
        }
    } else {
		ddata->state = RMC_PRELOAD;
    }
} break;
```
You can get the IR key value through DBG_CODE.

### Compie in IR Driver

Check the kernel config file. Make sure the following options are turned on:
```
CONFIG_ROCKCHIP_REMOTECTL=y
CONFIG_ROCKCHIP_REMOTECTL_PWM=y
```
Or use `make menuconfig` in kernel source tree, and select in the IR support:
```
Device Drivers
  --->Input device support
  ----->  [*]   rkxx remotectl
  ------->[*]   rkxx remoctrl pwm0 capture.
```  
Then you can make the kernel.

### Android Key Remapping

/system/usr/keylayout/20050030_pwm.kl is a file mapping scancodes in kernel to keycodes in Android. You can add or modify it to match your IR remote's keys.  
The content of the file is:
```
key 28    ENTER
key 116   POWER             WAKE
key 158   BACK
key 139   MfENU
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
key 67    TV_MEDIA_REWIND
key 68    TV_MEDIA_FAST_FORWARD
key 87    TV_MEDIA_PREVIOUS
key 88    TV_MEDIA_NEXT
key 250   FIREFLY_RECENT
```
You can modify this file via adb and reboot to take effect.
