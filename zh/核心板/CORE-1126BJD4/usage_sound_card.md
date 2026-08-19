## Sound Card 使用

### EarPhone && Speak
EarPhone 和 Speak 均采用双声道接口

使用 aplay 命令播放 wav 格式音频
```
#path-to 表示存放音频的绝对路径
aplay /path-to/audio-name.wav
```
如果ALSA没有设置默认设备或者配置错误，直接aplay是无法播放，需要指定正确的声卡才能播放
```
aplay -D hw:0,0 /path-to/audio.wav
```
如果想设置默认设备,可以在/etc/目前下创建asound.conf文件，输入以下内容。
```
pcm.!default {
    type plug
    slave {
        pcm "hw:0,0"
    }
}

ctl.!default {
    type hw
    card 0
}
```
### Mic

* RV1126B有ADC0(PMU)和ADC1，可支持到4通道录音。RV1126BJD4底板具有两个MIC接口。MIC2接到ADC0(PMU)，MIC1接到ADC1。
![](../../../rv1126b_img/CORE-1126BJD4/usage_sound.png)
* 固件默认开启4通道录音，2通道录音默认选择MIC2接口。录音功能默认未开启，需手动开启。
* 如下是开启ADC0即MIC2录音，如果再将ADC1开启，录音即可4通道录音，播放4通道音频需要播放设备支持4通道播放才可。
* MIC2开启
```bash
amixer -c 1 cset name='ACodec ADC Switch' 1						# 使能 ADC0
amixer -c 1 cset name='ACodec Digital Gain Volume' 127			# ADC0 的数字增益调到最大。127 最大，0 最小
amixer -c 1 cset name='ACodec PGA Gain Volume' 31				# ADC0 的模拟增益调到最大
```
* MIC1开启
```bash
amixer -c 1 cset name='ACodec_LP ADC Switch' 1                  # 使能 ADC1
amixer -c 1 cset name='ACodec_LP Digital Gain Volume' 127       # ADC1 的数字增益调到最大。127 最大，0 最小
amixer -c 1 cset name='ACodec_LP PGA Gain Volume' 31            # ADC1 的模拟增益调到最大
```
* 如果只想选择2通道录音，即在MIC1和MIC2两个之间选择一个录音2通道音频，需要手动切换
* 默认支持MIC2进行2通道录音，如果要切换到MIC1进行2通道录音，需切换PATH输入源
* MIC1 
```bash
amixer -c 1 cset name='SAI2 Receive PATH0 Source Select' 1 
amixer -c 1 cset name='SAI2 Receive PATH1 Source Select' 0 
```
* MIC2
```bash
amixer -c 1 cset name='SAI2 Receive PATH0 Source Select' 0      # 默认
amixer -c 1 cset name='SAI2 Receive PATH1 Source Select' 1      # 默认
```
* Mic 录制音频
```
arecord -l														# 查看所有可用的 MIC 设备。
arecord -Dhw:1,0 -f cd -d 10 /path-to/audio.wav					# 选择声卡并录制音频
```

更多关于 RV1126B acodec 的详细配置，请下载[《RV1126B acodec 配置文档》](https://www.t-firefly.com/doc/download/333.html#other_932) 进行查看。