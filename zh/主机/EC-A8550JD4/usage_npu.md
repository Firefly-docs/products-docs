# AI 教程

EC-A8550JD4 所采用的 QCS8550 处理器拥有 Hexagon 数据处理芯片(DSP)，里面包含用于处理 AI 计算的 NPU。

所以一般情况下 NPU 或 DSP 都是指这同一颗芯片，并不是两个不同的芯片。

## Aidlux 介绍

Aidlux 是一套完备的边缘端 AI 开发工具套件，能简化高通平台的环境部署，帮助开发者加速 AI 应用落地。

目前一般使用 docker 的方式部署 Aidlux Web Desktop 来进行开发，可通过如下命令查看设备上是否已存在名为 aidlux 的容器：

```bash
docker ps -a
```

Aidlux Web Desktop 在 EC-A8550JD4 上的预装与安装方法待补充。

## AI 开发

Aidlite 是 AI 执行框架，可以调用高通平台的 CPU/GPU/NPU(DSP) 进行模型加速推理；AidGenSE 是适配了 OpenAI HTTP 协议的生成式 AI HTTP 服务，开发者可以通过 HTTP 方式调用生成式 AI 并快速集成到自己的应用中。

Aidlux Web Desktop 的部署与 AI 开发的详细步骤，可前往 [Aidlux 文档中心](https://rhinopi.docs.aidlux.com/software/) 查看。