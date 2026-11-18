# 大语言模型部署

## 简介 [step]

Firefly-R3572A 基于 RK3572 NPU（RKNPU3）支持大语言模型（LLM）的板端私有化部署，覆盖 Qwen、GLM、MiniCPM、Gemma 等系列模型，适用于边缘侧离线对话、多模态理解等场景。

<Callout title="重要" type="warn">

RK3572 的大语言模型部署使用 RKNN3 SDK 体系中的 **RKLLM3**，与 RK3588/RK3576 等平台使用的 `rknn-llm`（RKLLM-Toolkit / `llm_demo`）**不同，不可混用**。RKNPU2 时代的 `rknn-llm`、`rkllm_api_demo`、`librknnrt.so` 不适用于 RK3572。

</Callout>

## RKLLM3 介绍 [step]

RKLLM3 是 RKNN3 SDK 提供的大语言模型部署方案，核心是 **rkllm3-server**——一套基于 RKNPU3 实现的 LLM Server，底层基于 [llama.cpp](https://github.com/ggml-org/llama.cpp)，提供 OpenAI API 兼容的对话接口。

* **模型推理**：在板端通过 RKNPU3 驱动执行 LLM 推理
* **对话接口**：OpenAI API 兼容的 REST 接口，便于上层应用对接
* **多模态支持**：支持 LLM + Vision（图像）、Audio（语音）等多模态模型
* **多卡级联**：支持多设备级联推理（`--stage-count`）

rkllm3-server 位于 SDK `external/rknn3-runtime/rkllm3-server/`，提供 Linux aarch64 与 Android arm64-v8a 的可执行程序及 `librkllm3-server.so`。

## 支持模型 [step]

RK3572 支持的大语言及多模态模型包括（以 SDK `external/rknn3-runtime/README_CN.md` 为准）：

| 类别 | 代表模型 |
| --- | --- |
| LLM | Qwen2.5 / Qwen3 / Qwen3.5、GLM-Edge、Youtu-LLM、MiniCPM5、FunctionGemma、LFM2.5 |
| VLM | Qwen2.5-VL / Qwen3-VL / Qwen3.5-VL、MiniCPM-V-4、InternVL3、SmolVLM2 |
| Omni | Qwen2.5-Omni / Qwen3-Omni、Gemma4 |
| ASR / TTS | Qwen3-ASR、Whisper、SenseVoice、Qwen3_TTS、VITS |
| Embedding / Reranker | Qwen3-Embedding、Qwen3-Reranker |

更多模型与预转换 RKNN 模型请参考 [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo) 及 SDK 网盘（预转换模型位于 `RKNN3_SDK/rknn3_models/v1.1.0`）。

## 获取模型 [step]

大语言模型的转换请参考 RKNN3-Toolkit 文档与 [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo) 中的 LLM 示例；也可直接从 SDK 网盘下载预转换好的 RKNN 模型（位于 `RKNN3_SDK/rknn3_models/v1.1.0`）。

一个完整的 LLM 模型通常包含以下文件，部署时需一并拷贝到主板：

* `*.rknn`：转换后的模型主体
* `*.gguf`（`--vocab`）：词汇表文件
* `*.embed.bin`（`--embed`）：词嵌入文件（tie-word 模式可不需要）
* 权重文件 `*.weight`（部分模型需要）

## 部署运行 [step]

将 rkllm3-server（或 `librkllm3-server.so`）、模型与相关文件拷贝到主板后，运行服务。以下为 SDK README 的标准模式示例（路径按实际修改）：

```bash
# 位于 SDK external/rknn3-runtime/rkllm3-server/bin/linux-aarch64/
# LLM 模型（标准模式，需要 embed 文件）
./rkllm3-server -m qwen2.5-3b.rknn \
    --vocab qwen2.5-3b.tokenizer.gguf \
    --embed qwen2.5-3b.embed.bin \
    --host 0.0.0.0 --port 8080 -c 768 --n_predict 512
```

tie-word 模式（不需要 embed 文件）示例：

```bash
./rkllm3-server -m Qwen3-VL-2B-llm-tie-word.rknn \
    --weight Qwen3-VL-2B-llm-tie-word.weight \
    --vocab Qwen3-VL-2B-llm_quant.tokenizer.gguf \
    --tie-word-embedding 1 --host 0.0.0.0 --port 8080 -c 4096 --n_predict 512
```

服务启动后监听 `--host` / `--port`（默认 127.0.0.1:8080），通过 OpenAI API 兼容的 REST 接口进行对话。

<Callout title="注意" type="info">

以上是命令参数示例，实际模型文件名、参数与文件组合以你手中 SDK 的 rkllm3-server README 与 `rknn3-model-zoo` 对应模型目录为准。可通过 `./rkllm3-server --help` 查看全部参数（如 `--core-mask` 核心掩码、`--temp`/`--top-k`/`--top-p` 采样参数等）。

</Callout>

## NPU 驱动要求 [step]

Firefly-R3572A 使用 **RKNPU3 驱动**（设备节点 `/dev/rknpu3`，sysfs 目录 `/sys/class/misc/rknpu3`），当前固件驱动版本为 v1.0.3。运行 RKLLM3 前可确认驱动已正常加载：

```shell
root@firefly:/# dmesg | grep -i RKNPU3
[    4.692690] RKNPU3 26b00000.npu: RKNPU3 Driver v1.0.3 (20260613) loaded successfully
root@firefly:/# ls /dev/rknpu3
/dev/rknpu3
```

<Callout title="注意" type="info">

`cat /sys/kernel/debug/rknpu/version` 是旧版 rknpu 驱动的接口，RK3572 实际使用 RKNPU3 驱动，无此 debugfs 路径；请改用上面的 `dmesg` 方式查看版本。

</Callout>

## FAQs [step]

**Q1：服务启动后模型加载失败？**

A1：确认模型是 RKNN3-Toolkit 转换（与 RKNN3 Runtime 版本匹配）、`--vocab`/`--embed`/`--weight` 等文件齐全且路径正确；同时确认 `/dev/rknpu3` 存在、RKNPU3 驱动已加载。

**Q2：是否可以使用 RK3588 的 rknn-llm / llm_demo 模型？**

A2：不可以。RK3588/RK3576 使用 RKNPU2 + rknn-llm，与 RK3572 的 RKNPU3 + RKLLM3 是两套体系，工具链与模型格式均不兼容。请使用 RKNN3-Toolkit 转换模型或下载 RKNN3 预转换模型。

**Q3：转换大模型时 PC 内存不足？**

A3：参数量越大的模型转换所需内存越大，可增大 swapfile 或使用大内存的 PC。