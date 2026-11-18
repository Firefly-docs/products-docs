# Large Language Model Deployment

## Introduction [step]

Firefly-R3572A, based on the RK3572 NPU (RKNPU3), supports on-board private deployment of large language models (LLMs), covering Qwen, GLM, MiniCPM, Gemma and other series, suitable for edge-side offline conversation, multimodal understanding and other scenarios.

<Callout title="Important" type="warn">

LLM deployment on RK3572 uses **RKLLM3** from the RKNN3 SDK, which is **different from and incompatible with** the `rknn-llm` (RKLLM-Toolkit / `llm_demo`) used on RK3588/RK3576. The RKNPU2-era `rknn-llm`, `rkllm_api_demo` and `librknnrt.so` do not apply to RK3572.

</Callout>

## RKLLM3 Overview [step]

RKLLM3 is the LLM deployment solution in the RKNN3 SDK. Its core is **rkllm3-server**, an LLM server based on RKNPU3 and [llama.cpp](https://github.com/ggml-org/llama.cpp), providing an OpenAI API-compatible chat interface.

* **Model inference**: runs LLM inference on-board through the RKNPU3 driver
* **Chat interface**: OpenAI API-compatible REST interface for easy integration
* **Multimodal support**: supports LLM + Vision (image) and Audio (speech) multimodal models
* **Multi-card cascading**: supports multi-device cascaded inference (`--stage-count`)

rkllm3-server is located in the SDK `external/rknn3-runtime/rkllm3-server/`, providing executables for Linux aarch64 and Android arm64-v8a as well as `librkllm3-server.so`.

## Supported Models [step]

The LLMs and multimodal models supported by RK3572 include (refer to the SDK `external/rknn3-runtime/README_CN.md`):

| Category | Representative models |
| --- | --- |
| LLM | Qwen2.5 / Qwen3 / Qwen3.5, GLM-Edge, Youtu-LLM, MiniCPM5, FunctionGemma, LFM2.5 |
| VLM | Qwen2.5-VL / Qwen3-VL / Qwen3.5-VL, MiniCPM-V-4, InternVL3, SmolVLM2 |
| Omni | Qwen2.5-Omni / Qwen3-Omni, Gemma4 |
| ASR / TTS | Qwen3-ASR, Whisper, SenseVoice, Qwen3_TTS, VITS |
| Embedding / Reranker | Qwen3-Embedding, Qwen3-Reranker |

For more models and pre-converted RKNN models, refer to [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo) and the SDK network drive (pre-converted models are under `RKNN3_SDK/rknn3_models/v1.1.0`).

## Getting the Model [step]

For model conversion, refer to the RKNN3-Toolkit documentation and the LLM examples in [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo). Alternatively, download pre-converted RKNN models from the SDK network drive (under `RKNN3_SDK/rknn3_models/v1.1.0`).

A complete LLM model usually consists of the following files, which should be copied to the board together:

* `*.rknn`: the converted model body
* `*.gguf` (`--vocab`): the vocabulary file
* `*.embed.bin` (`--embed`): the embedding file (not required in tie-word mode)
* A weight file `*.weight` (required by some models)

## Deployment and Running [step]

Copy rkllm3-server (or `librkllm3-server.so`), the model and related files to the board, then start the server. The following is the standard-mode example from the SDK README (modify the paths accordingly):

```bash
# Located in SDK external/rknn3-runtime/rkllm3-server/bin/linux-aarch64/
# LLM model (standard mode, requires an embed file)
./rkllm3-server -m qwen2.5-3b.rknn \
    --vocab qwen2.5-3b.tokenizer.gguf \
    --embed qwen2.5-3b.embed.bin \
    --host 0.0.0.0 --port 8080 -c 768 --n_predict 512
```

Example in tie-word mode (no embed file required):

```bash
./rkllm3-server -m Qwen3-VL-2B-llm-tie-word.rknn \
    --weight Qwen3-VL-2B-llm-tie-word.weight \
    --vocab Qwen3-VL-2B-llm_quant.tokenizer.gguf \
    --tie-word-embedding 1 --host 0.0.0.0 --port 8080 -c 4096 --n_predict 512
```

Once started, the server listens on `--host` / `--port` (default 127.0.0.1:8080) and provides chat through the OpenAI API-compatible REST interface.

<Callout title="Note" type="info">

The commands above are parameter examples; the actual model file names, parameters and file combinations depend on the rkllm3-server README and the corresponding model directory in `rknn3-model-zoo` in your SDK. Run `./rkllm3-server --help` for the full list of parameters (such as the `--core-mask` core mask and sampling parameters `--temp`/`--top-k`/`--top-p`).

</Callout>

## NPU Driver Requirements [step]

Firefly-R3572A uses the **RKNPU3 driver** (device node `/dev/rknpu3`, sysfs directory `/sys/class/misc/rknpu3`), with the current firmware shipping driver v1.0.3. Before running RKLLM3, confirm the driver is loaded:

```shell
root@firefly:/# dmesg | grep -i RKNPU3
[    4.692690] RKNPU3 26b00000.npu: RKNPU3 Driver v1.0.3 (20260613) loaded successfully
root@firefly:/# ls /dev/rknpu3
/dev/rknpu3
```

<Callout title="Note" type="info">

`cat /sys/kernel/debug/rknpu/version` belongs to the legacy rknpu driver; the RK3572 uses the RKNPU3 driver and has no such debugfs path. Use the `dmesg` method above instead.

</Callout>

## FAQs [step]

**Q1: The model fails to load after starting the server?**

A1: Make sure the model was converted with RKNN3-Toolkit (matching the RKNN3 Runtime version), that the `--vocab`/`--embed`/`--weight` files are complete and correctly pathed, and that `/dev/rknpu3` exists with the RKNPU3 driver loaded.

**Q2: Can I use RK3588's rknn-llm / llm_demo models?**

A2: No. RK3588/RK3576 use RKNPU2 + rknn-llm, which is a different stack from RK3572's RKNPU3 + RKLLM3; the toolchains and model formats are incompatible. Use RKNN3-Toolkit to convert models or download pre-converted RKNN3 models.

**Q3: Insufficient PC memory when converting a large model?**

A3: Larger models require more memory to convert. Increase the swapfile or use a PC with more memory.