# K3 AI Usage

For AI usage on the K3 Bianbu system, please click the link: [AI Tutorial](https://spacemit.com/community/document/info?nodepath=ai/intro/root_overview.md&lang=en)

## AI SDK Introduction

The SpacemiT AI SDK is an AI application development kit for the K-series chips. It supports the Buildroot and Bianbu LXQT/GNOME systems on K1/K3. Its main components include:

* **Vision**: vision tasks including detection, classification, segmentation, tracking, face and pose, with support for models such as resnet, yolov8/yolov11, etc.
* **Speech**: VAD (voice activity detection), ASR (speech recognition), TTS (speech synthesis) and voiceprint capabilities
* **LLM**: large language model inference, providing an OpenAI-compatible llama-server interface
* **VLM**: vision-language models for image-text understanding, supporting FastVLM and the Qwen series
* **RL**: robot policy inference
* **Gateway**: unified HTTP/WS service access layer

## Compute Overview

The **AIBOX-K3** is powered by the SpacemiT Key Stone K3 SoC, which adopts a homogeneous RISC-V fusion compute architecture. It integrates 8 high-performance general-purpose X100 cores and 8 ultra-wide parallel AI A100 cores, delivering 130K DMIPS of general-purpose compute and 60 TOPS of general AI compute. It supports running large language models up to 30B parameters and is compatible with rapid deployment of all kinds of AI algorithms and models.

## Installation and Build

```sh
git clone --recurse-submodules https://github.com/spacemit-com/ai-sdk.git
source build/envsetup.sh
m
```

* The first build takes a long time, please be patient
* Build artifacts are installed to `output/staging`
* To build a single component: enter the corresponding component directory and run `mm`
* The Gateway can also be installed directly: `sudo apt install spacemit-ai-gateway` (the backend uses port 18790 by default, and the console uses port 8326)

## Common Examples

* **Vision**: first run `vision/scripts/download_all_models.sh` and `download_assets.sh` to download models and assets, then run an example, e.g. yolov8:

    ```sh
    yolov8 vision/examples/yolov8/config/yolov8.yaml
    ```

* **LLM**: after downloading a GGUF model, start a local service with `llama-server` (port 8080 by default) and chat with `llm_chat`; for cloud services, change the api_base and `export OPENAI_API_KEY`
* **ASR/TTS/VAD**: run `asr_file_demo`, `tts_file_demo` and `vad_simple_demo` respectively
* **Gateway verification**: `curl localhost:18790/healthz`

## FAQ

* Only one model service can run on the same port (e.g. 8063); stop the current service before switching models
* The bytetrack/ocsort real-time tracking examples require an external display
* After a VLM test is interrupted, clean up leftover processes and NPU state with `pkill -f llama-server` and `spacemit-tcm-smi -c`
* For detailed parameters and examples, see the `vision/examples/*/README.md` of each component

## More Resources

* [AI Tutorial](https://spacemit.com/community/document/info?nodepath=ai/intro/root_overview.md&lang=en): Official AI documentation entry, covering AI frameworks, deployment methods and examples
* [Chip Product Documents](https://spacemit.com/community/document/info?lang=en&nodepath=hardware/key_stone/k3/k3_docs): including Product Introduction, Datasheet and User Manual