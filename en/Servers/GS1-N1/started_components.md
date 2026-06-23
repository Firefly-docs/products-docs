
# Introduction
GS1-N1 High-performance dual-core board module design, It adopts dual-core heterogeneous architecture. One core is responsible for video
recording, and the other is for AI processing. It supports real-time inference of AI models.
The two core modules work independently, optimizing resource allocation and enabling
parallel processing of multiple tasks, which significantly improves the system performance

## Main mod

|Optional mods  |Memory |Storage |AI perf. |
|----|----|----|----|
|RK3576(Core-3576JD4) |LPDDR4/LPDDR4x(2/4/8/16GB) |eMMC(16/32/64/128/256GB)|6T|
|RK3588(Core-3588JD4) |LPDDR4/LPDDR4x(4/8/16/32GB)|eMMC(32/64/128/256GB)   |6T|
|SOPHGO(Core-1688JD4) |LPDDR4(4/8/16GB)           |eMMC(32/64/128/256GB)   |16T|
|NVIDIA(Jetson Orin Nano) |LPDDR5 (8GB)           |Not                      |67T|
|NVIDIA(Jetson Orin NX) |LPDDR5 (16GB)           |Not                      |157T|

|Optional mods |Video decoding |Video encoding |
|----|----|----|
|RK3576(Core-3576JD4)    |8K@30fps H.265/VP9/AVS2/AV1<br>4K@120fps H.265/VP9/AVS2/AV1<br>4K@60fps H.264/AVC|4K@60fps H.265/H.264|
|RK3588(Core-3588JD4)    |8K@60fps H.265/VP9/AVS2<br>8K@30fps H.264 AVC/MVC<br>4K@60fps AV1<br>1080P@60fps MPEG-2/-1/VC-1/VP8|8K@30fps H.265/H.264|
|SOPHGO(Core-1688JD4)    |8K@30fps H.265/H.264|8K@15fps H.265/H.264|
|NVIDIA(Jetson Orin Nano)|1x 4K60, 2x 4K30, 5x 1080p60 H.265|1080p30 supported by 1-2 CPU cores|
|NVIDIA(Jetson Orin NX)  |1x 8K30, 2x 4K60, 9x 1080p60 H.265|1x 4K60, 3x 4K30, 6x 1080p60 H.265|

## Sub mod
* You can choose any of the main mods
* The sub mod can be empty
