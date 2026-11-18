# NPU Usage

## Introduction [step]

The RK3572 in Firefly-R3572A has a built-in NPU with 4 TOPS (sparse INT8) of computing power, supporting INT4/INT8/INT16/FP4/FP8/FP16/BF16 mixed operations and W4A16 asymmetric MAC operations. It can be used for edge-side inference of CNN, YOLO object detection and large language models.

<Callout title="Important" type="warn">

The RK3572 NPU uses the new **RKNN3 SDK** software stack and is **incompatible with RKNN-Toolkit2 / RKNPU2**. The `rknn-toolkit2`, `librknnrt.so` and `rknn_server` used by RK3588/RK3576/RK3568 (the RKNPU2 stack) **do not apply to RK3572**. Use the RKNN3-Toolkit and RKNN3-Runtime described in this chapter.

</Callout>

## RKNN3 SDK Overview [step]

RKNN3 SDK provides the complete software stack for deploying AI models to RK1820/RK1828/**RK3572**, including:

* **RKNN3-Toolkit**: a PC development toolkit for model conversion, quantization, inference and performance evaluation
* **RKNN3-Runtime**: the board-side runtime library (`librknn3_api.so`) providing C/C++ APIs to load RKNN models and run inference on the NPU
* **RKNN3 Model Zoo**: conversion and deployment examples for CNN / LLM / VLM models

The components are listed below:

| Component | Description | Location |
| --- | --- | --- |
| RKNN3-Toolkit | PC model conversion tool (Python 3.10 / 3.12) | SDK `external/rknn3-toolkit/`; open source at [rknn3-toolkit](https://github.com/airockchip/rknn3-toolkit) |
| RKNN3-Runtime | Board-side runtime library and header `rknn3_api.h` | SDK `external/rknn3-runtime/rknn3-api/` |
| RKNN3 Model Zoo | Model conversion and deployment examples | [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo) |

The current SDK version is **RKNN3 SDK V1.1.0**, and the board-side runtime build package version is `RKNN3_RUNTIME_VERSION = 1.0.0`. Refer to `external/rknn3-runtime/README_CN.md` and the release notes in the SDK for details on supported platforms and versions.

## NPU Driver and Device Node [step]

The RK3572 in Firefly-R3572A uses the **RKNPU3 kernel driver** (device node `/dev/rknpu3`, sysfs directory `/sys/class/misc/rknpu3`), built with `CONFIG_ROCKCHIP_RKNPU3=y`. Check the driver version and hardware information from the kernel log:

```bash
# Check the RKNPU3 driver version (e.g. RKNPU3 Driver v1.0.3 (20260613))
dmesg | grep -i RKNPU3

# Check the NPU device node and sysfs directory
ls -l /dev/rknpu3
ls /sys/class/misc/rknpu3/

# Check the current NPU frequency
cat /sys/class/misc/rknpu3/device/devfreq/*/cur_freq
```

<Callout title="Note" type="info">

The commonly seen `cat /sys/kernel/debug/rknpu/version` belongs to the legacy **rknpu** driver's debugfs interface. The RK3572 actually binds to the RKNPU3 driver (`/sys/class/misc/rknpu3`, `/dev/rknpu3`) and has no such debugfs path. Use the `dmesg` method above to check the driver version.

</Callout>

## Board-side Runtime Environment [step]

RKNN3-Runtime provides board-side C/C++ APIs. The core header is `rknn3_api.h` and the library is `librknn3_api.so`, located under `external/rknn3-runtime/rknn3-api/` by target architecture (e.g. `Linux/aarch64/`, `Android/arm64-v8a/`).

* Header: `rknn3-api/include/rknn3_api.h`
* Library: `rknn3-api/Linux/aarch64/librknn3_api.so` (Linux), `rknn3-api/Android/arm64-v8a/librknn3_api.so` (Android)

Add the header and library paths to your project and link against `librknn3_api.so` to use the RKNN3 Runtime APIs. Build and deployment examples are available in the SDK `external/rknn3-test/` (`build-linux.sh`, `build-android.sh` and the CNN/LLM tests under `rknn3_stress_test/`).

<Callout title="Note" type="warn">

RKNN3-Toolkit, RKNN3-Runtime and the firmware NPU driver must be version-matched. The current Firefly-R3572A firmware ships the RKNPU3 driver v1.0.3 with RKNN3 SDK V1.1.0. Do not mix in libraries or models from the legacy RKNN-Toolkit2 / RKNPU2 stack; otherwise models may fail to load or produce inference errors.

</Callout>

## Model Conversion [step]

Install RKNN3-Toolkit on a PC (Ubuntu, Python 3.10 or 3.12) and convert the trained model to RKNN format. For the installation procedure, APIs and conversion parameters, refer to the documentation in the SDK `external/rknn3-toolkit/` (CN/EN) and [rknn3-toolkit](https://github.com/airockchip/rknn3-toolkit).

For conversion and deployment examples, refer to RKNN3 Model Zoo ([rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo)), which provides complete conversion and board-side deployment examples for CNN, LLM, VLM and other models. Pre-converted RKNN models are also available from the SDK network drive (under `RKNN3_SDK/rknn3_models/v1.1.0`).

## On-board Inference [step]

After deploying the converted `.rknn` model and the RKNN3-Runtime library to the board, use the APIs provided by `rknn3_api.h` to load the model, feed input data and obtain inference results. The SDK `external/rknn3-test/rknn3_stress_test/` contains ready-to-build CNN test programs that can be used as a reference.

For large language model (LLM) deployment, RKLLM3 is used; see [Large Language Model](usage_rkllm.md).

## Verify Deployment [step]

Run an RKNN3 example from the SDK (e.g. the CNN test built from `external/rknn3-test/`) to confirm the model loads and runs correctly. If initialization fails, check the following in order:

1. Whether `/dev/rknpu3` exists and the RKNPU3 driver is loaded (`dmesg | grep -i RKNPU3`);
2. Whether the model was converted with RKNN3-Toolkit and matches the board-side Runtime version;
3. Whether libraries or models from the RKNPU2 / RKNN-Toolkit2 stack were used by mistake.

## Detailed Documentation [step]

For detailed usage of the RK3572 NPU and RKNN3 SDK, refer to the documentation under SDK `external/rknn3-runtime/doc/` and `external/rknn3-toolkit/doc/`, as well as the [RKNN3-Toolkit](https://github.com/airockchip/rknn3-toolkit) and [RKNN3-Model-Zoo](https://github.com/airockchip/rknn3-model-zoo) open-source repositories.