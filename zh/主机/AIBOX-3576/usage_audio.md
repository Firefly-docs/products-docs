# Audio 
AIBOX-3576 拥有两路音频输出以及一路音频输入。


## 音频输出

用户可以通过耳机口以及 HDMI 口去做音频的输出。

### 界面切换

在系统设置中切换耳机以及 HDMI 接口，选择单独一路进行输出：

![](../../../aibox_img/AIBOX-3576/Sound.png)

### 命令行模式

在终端下，执行命令：

```shell
# cat /proc/asound/cards
 0 [HDA            ]: tegra-hda - NVIDIA Jetson Orin Nano HDA
                      NVIDIA Jetson Orin Nano HDA at 0x3518000 irq 120
 1 [APE            ]: tegra-ape - NVIDIA Jetson Orin Nano APE
                      NVIDIA-NVIDIAJetsonOrinNanoDeveloperKit-NotSpecified-Jetson
```

* `HDA` 代表的是 HDMI 的声卡，设备号为 `3`
* `APE` 代表的是耳机的声卡，设备号为 `0`

* HDMI 

    ```shell
    aplay -D hw:HDA,3 hdmi.wav # 其中 hdmi.wav 需要双声道格式文件 
    ```

* 耳机

    ```shell
    amixer -c APE cset name="I2S2 Mux" ADMAIF1
    aplay -D hw:APE,0 test.wav
    ```

## 音频输入

同样是 `APE` 声卡，接入耳机开始录音，命令行执行：

```shell
amixer -c APE cset name="ADMAIF1 Mux" I2S2
arecord -D hw:APE,0 -c 2 -r 16000  -f S32_LE test.wav
```