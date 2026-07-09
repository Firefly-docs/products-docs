## Sound Card 使用

### Mic
* 录制音频

```bash
# 查看所有可用的MIC设备
$ arecord -l		

# 选择声卡并录制音频
$ arecord -f cd -Dhw:0,0 -d 10 -c 8 /tmp/test.wav
```

PC 使用 ADB 工具拷贝录音音频到电脑进行播放验证

```bash
adb pull /tmp/test.wav ./
```
