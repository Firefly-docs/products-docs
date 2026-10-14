# 音频教程

AIO-8550JD4 有 1 路 I2S 接入了 HDMI 用于音频播放。另一路 I2S 接入了底板声卡（耳机孔）

## 底层播放

底层仅能播放 48K 采样率 16 bit 的 wav 音频。

需要使用工具 agmplay

```bash
# 设置声卡录音通路
amixer -q set "PGAL Select" "Line 2P"
amixer -q set "PGAR Select" "Line 2N"

# 设置录音音量
amixer -q set "ADCL" "200"
amixer -q set "ADCR" "200"

# 录音 5 秒
agmcap -D 100 -d 101 -i MI2S-LPAIF-TX-PRIMARY -dkv 0xA3000004  -c 2 -r 48000 -b 16 -T 5 test.wav

# 设置播放音量
amixer -q set "DACL" "200"
amixer -q set "DACR" "200"

# 向耳机孔播放声音
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-PRIMARY -dkv 0xA2000001

# 向 HDMI 播放声音
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

## 上层播放

开发中...敬请期待