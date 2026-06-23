# 音频接口测试

AIBOX-3576  支持美标耳机接口。

播放测试：
```
$ aplay -D hw:CARD=cv186xdac,DEV=1 /usr/share/sounds/alsa/Front_Center.wav
```
录音测试：
```
$ arecord -D hw:CARD=cv186xadc,DEV=0  -c 2 -r 48000  -f S16_LE test.wav # 录制音频（单声道）
$ aplay -D hw:CARD=cv186xdac,DEV=1  test.wav # 播放录制好的音频
```