# USB升级

## 烧录固件

<font color="red">**注意：**</font>
* PC 端推荐 X86 Ubuntu 22.04 或者 Ubuntu 20.04，不推荐使用虚拟机。
* 烧录过程中会格式化 AIBOX-Orin NX 内置的存储设备，重要数据请先备份。
* Orin NX 电源适配器需要 **12V/5A**
* Orin NX 的底板硬件版本至少是 **V1.2**

### 下载 fireflyFlash.tbz2
[下载地址](https://community.t-firefly.com/download/237)
`固件` --> `Jetson Linux`

### 解压 fireflyFlash.tbz2
```
mkdir fireflyFlash
tar xf fireflyFlash.tbz2 -C fireflyFlash
cd fireflyFlash
sudo ./l4t_flash_prerequisites.sh
```

### AIBOX-Orin NX 进入烧录模式
* AIBOX-Orin NX 先断电
* 使用 Type-C 数据线连接 AIBOX-Orin NX 的 OTG 口和 PC 端
* 长按 AIBOX-Orin NX 的 Recovery 键
* AIBOX-Orin NX 供电
* 释放 AIBOX-Orin NX 的 Recovery 键
* 检查 AIBOX-Orin NX 是否进入 Recovery 模式
    * 使用 `lsusb` 命令， 当你看到 `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.` 时，即进入了 Recovery 模式。
        * `<bbb>` 任何三位数
        * `<ddd>` 任何三位数
        * `<nnnn>` 四位数
            * `7523` Jetson Orin Nano 8GB
            * `7423` Jetson Orin NX 8GB
            * `7323` Jetson Orin NX 16GB

### 烧录
在 `fireflyFlash` 目录下，执行命令：  `./firefly_flash.sh -d aibox`  

<font color=red>注意：烧录完，设备进入桌面后至少 5 分钟才能断电。</font>