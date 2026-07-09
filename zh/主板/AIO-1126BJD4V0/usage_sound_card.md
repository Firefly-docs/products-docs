## Sound Card 使用

### EarPhone && Speak
EarPhone 和 Speak 均采用双声道接口

使用 aplay 命令播放 wav 格式音频
```
#path-to 表示存放音频的绝对路径
aplay /path-to/audio-name.wav
```
### Mic

* Mic 使用到的是芯片的 ADC1 外设。默认是关闭的，使用时需要将其打开，通过以下操作进行

```bash
amixer -c 0 cset name='ACodec ADC Switch' 1						# 使能 ADC1
amixer -c 0 cset name='ACodec Digital Gain Volume' 0			# ADC1 的数字音量调到最大。0 最大，127 最小
amixer -c 0 cset name='ACodec PGA Gain Volume' 31				# ADC1 的数字音量调到最大

amixer -c 0 cset name='SAI2 Receive PATH0 Source Select' 1
amixer -c 0 cset name='SAI2 Receive PATH1 Source Select' 0
```
* Mic 录制音频
```
arecord -l														# 查看所有可用的 MIC 设备。
arecord -Dhw:0,0 -f cd -d 10 /path-to/audio.wav					# 选择声卡并录制音频
```

更多关于 RV1126B acodec 的详细配置，请下载[《RV1126B acodec 配置文档》](https://www.t-firefly.com/doc/download/333.html#other_932) 进行查看。