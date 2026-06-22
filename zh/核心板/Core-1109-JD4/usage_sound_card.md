## Sound Card 使用

### EarPhone && Speak
EarPhone 和 Speak 均采用双声道接口

使用 aplay 命令播放 wav 格式音频
```
#path-to 表示存放音频的绝对路径
aplay /path-to/audio-name.wav
```
### Mic
* Mic 默认是关闭的，使用时需要将其打开，通过以下操作进行
```
#命令设置默认录制声卡通道
amixer cset numid=2,iface=MIXER,name='Capture MIC Path' 1

#图形界面设置默认录制声卡通道
alsamixer
#将 Capture MIC Path 设置为 Main Mic
```
* Mic录制音频
```
arecord -l											#查看所有可用的MIC设备
arecord -Dhw:0,0 -f cd -d 10 /path-to/audio.wav	#选择声卡并录制音频
#注：Mic 只支持单声道录音
```