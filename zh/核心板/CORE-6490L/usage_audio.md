# 音频教程

Firefly-Q6490A 使用了一颗 ES8390 作为声卡。输出通道和输入通道都各有两路。

* INPUT1 <- 耳机口录音
* INPUT2 <- 拓展排针，用于外接 line in
* OUTPUT1 -> 拓展排针，用于外接 line out
* OUTPUT2 -> 耳机口播放

系统使用的声音服务为 ALSA + Pipewire

## 播放
```bash
# 可以先设置音量
# DACL 和 DACR 分别为左声道和右声道的音量，取值范围是 0~255
amixer -q -D hw:QCM6490IDP set DACL 200
amixer -q -D hw:QCM6490IDP set DACR 200

# 播放
pw-play test.wav
pw-play test.mp3
```

## 录音
```bash
# 切换输入通道为通道 1
amixer -q -D hw:QCM6490IDP set "PGAL Select" "Line 1P"
amixer -q -D hw:QCM6490IDP set "PGAR Select" "Line 1N"

# 设置录音增益，取值范围是 0~255
amixer -q -D hw:QCM6490IDP set ADCL 200
amixer -q -D hw:QCM6490IDP set ADCR 200

# 开始录音
pw-record output.wav

# 播放录音
pw-play output.wav
```