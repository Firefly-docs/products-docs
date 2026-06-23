# 音频教程

AIBOX-8550 有 1 路 I2S 接入了 HDMI 用于音频播放。

## 底层播放

底层仅能播放 48K 采样率 16 bit 的 wav 音频。

需要使用工具 agmplay

```bash
# 向 HDMI 播放声音
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

## 上层播放

开发中...敬请期待