
# 简介
GS1-N1 高性能双核心板模组设计, 采用双核心异构，一个负责视频录像、一个负责 AI 处理，支持 AI 模型实时推理，两核心模组独立工作，多任务并行 。 

## 主模组

|可选模组 |内存 |存储 |AI性能 |
|----|----|----|----|
|RK3576(Core-3576JD4) |LPDDR4/LPDDR4x(2/4/8/16GB) |eMMC(16/32/64/128/256GB)|6T|
|RK3588(Core-3588JD4) |LPDDR4/LPDDR4x(4/8/16/32GB)|eMMC(32/64/128/256GB)   |6T|
|SOPHGO(Core-1688JD4) |LPDDR4(4/8/16GB)           |eMMC(32/64/128/256GB)   |16T|
|NVIDIA(Jetson Orin Nano) |LPDDR5 (8GB)           |无                      |67T|
|NVIDIA(Jetson Orin NX) |LPDDR5 (16GB)           |无                      |157T|

|可选模组 |视频解码 |视频编码 |
|----|----|----|
|RK3576(Core-3576JD4)    |8K@30fps H.265/VP9/AVS2/AV1<br>4K@120fps H.265/VP9/AVS2/AV1<br>4K@60fps H.264/AVC|4K@60fps H.265/H.264|
|RK3588(Core-3588JD4)    |8K@60fps H.265/VP9/AVS2<br>8K@30fps H.264 AVC/MVC<br>4K@60fps AV1<br>1080P@60fps MPEG-2/-1/VC-1/VP8|8K@30fps H.265/H.264|
|SOPHGO(Core-1688JD4)    |8K@30fps H.265/H.264|8K@15fps H.265/H.264|
|NVIDIA(Jetson Orin Nano)|1x 4K60, 2x 4K30, 5x 1080p60 H.265|1080p30，由 1-2 个 CPU 核心提供支持|
|NVIDIA(Jetson Orin NX)  |1x 8K30, 2x 4K60, 9x 1080p60 H.265|1x 4K60, 3x 4K30, 6x 1080p60 H.265|

## 副模组
* 可选主模组里任意一款
* 副模组可为空
