# 图片内容检索

## CLIP-TPU 项目简介

CLIP 全称 Constrastive Language-Image Pre-training，是 OpenAI 推出的采用对比学习的文本-图像预训练模型。CLIP 在 zero-shot 文本-图像检索，zero-shot 图像分类，文生图任务 guidance，open-domain 检测分割等任务上均有非常惊艳的表现。

CLIP-TPU 项目实现了对 CLIP 的 TPU 算法移植，用户可使用该项目验证 CLIP 在 AIO-1684XQ 上的实际表现。

项目仓库链接：[CLIP-TPU](https://github.com/wangyifan2018/CLIP-TPU)

## ImageSearch-tpu 项目简介

ImageSearch-tpu 项目基于 CLIP 实现了对大量图片进行内容搜索的功能，同时提供了可视化的 WebUI 便于用户使用。

项目仓库链接：[ImageSearch-tpu](https://github.com/wangyifan2018/ImageSearch-tpu)