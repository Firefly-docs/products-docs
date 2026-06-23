# Audio 
## 音频输出

用户可以通过耳机口以及 HDMI 口去做音频的输出。

### 界面切换

在系统设置中切换耳机以及 HDMI 接口，选择单独一路进行输出。

### 命令行模式

在终端下，执行命令： `cat /proc/asound/cards`
* `HDA` 代表的是 HDMI 的声卡
* `APE` 代表的是耳机的声卡

#### HDMI 
```shell
aplay -D hw:HDA,3 hdmi.wav # 其中 hdmi.wav 需要双声道格式文件 
```

#### 耳机
```shell
aplay -D hw:APE,0 test.wav
```

## 音频输入

同样是 `APE` 声卡，接入耳机开始录音，命令行执行：

```shell
arecord -D hw:APE,0 -c 2 -r 16000  -f S32_LE test.wav
```

