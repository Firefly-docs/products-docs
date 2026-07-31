
# Introduction
RK182X Developer Kit High-performance dual-core board module design, It adopts dual-core heterogeneous architecture. One core is responsible for video
recording, and the other is for AI processing. It supports real-time inference of AI models.
The two core modules work independently, optimizing resource allocation and enabling
parallel processing of multiple tasks, which significantly improves the system performance

## Main mod

|Optional mods  |Memory |Storage |AI perf. |
|----|----|----|----|
|Core-3588JD4 |LPDDR4/LPDDR4x(4/8/16/32GB)|eMMC(32/64/128/256GB)   |6T|
|Core-3588SJD4 AI |LPDDR4/LPDDR4x(4/8/16/32GB)|eMMC(32/64/128/256GB)   |6T|
|Core-3576JD4 |LPDDR4/LPDDR4x(4/8/16/32GB)|eMMC(32/64/128/256GB)   |6T|

|Optional mods |Video decoding |Video encoding |
|----|----|----|
|Core-3588JD4 |8K@60fps H.265/VP9/AVS2<br>8K@30fps H.264 AVC/MVC<br>4K@60fps AV1<br>1080P@60fps MPEG-2/-1/VC-1/VP8|8K@30fps H.265/H.264|
|Core-3588SJD4 AI |8K@60fps H.265/VP9/AVS2<br>8K@30fps H.264 AVC/MVC<br>4K@60fps AV1<br>1080P@60fps MPEG-2/-1/VC-1/VP8|8K@30fps H.265/H.264|
|Core-3576JD4 |8K@30fps H.265/VP9/AVS2/AV1<br>4K@120fps H.265/VP9/AVS2/AV1<br>4K@60fps H.264/AVC|4K@60fps H.265/H.264|

## Sub mod
* RK1820 SODIMM 
    * DDR: 2.5GB
    * PCIe2.1: 1 Lane, EP
    * NPU: INT4/INT8/INT16/FP8/FP16/BF16, 20 TOPS for INT8
* RK1828 SODIMM
    * DDR: 5GB
    * PCIe2.1: 1 Lane, EP
    * NPU: INT4/INT8/INT16/FP8/FP16/BF16, 20 TOPS for INT8
