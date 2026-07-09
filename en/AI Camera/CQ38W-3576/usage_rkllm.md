# Large Language Model
## RKLLM Introduction
The RKLLM SDK helps users quickly deploy large language models onto CQ38W-3576.
[SDK Download](https://github.com/airockchip/rknn-llm)

### RKLLM-Toolkit Functions Introduction
The RKLLM-Toolkit is a development suite designed for users to perform quantization and conversion of large language models on their computers. Through the Python interface provided by this tool, users can conveniently achieve the following functions:
* Model Conversion: Supports converting large language models in Hugging Face format to RKLLM models. The converted RKLLM models can be loaded and used on the Rockchip NPU platform.
* Quantization: Supports quantizing floating-point models to fixed-point models. Currently supported quantization types include w4a16 and w8a8.

### RKLLM Runtime Functions Introduction
The RKLLM Runtime is primarily responsible for loading RKLLM models converted using the RKLLM-Toolkit and performing inference on the Rockchip NPU by invoking the NPU driver on the CQ38W-3576 board. During the inference of RKLLM models, users can customize the inference parameters, define various text generation methods, and continuously receive inference results through predefined callback functions.

## RKLLM-Toolkit Installation
The RKLLM-Toolkit is currently only available for Linux PC, with `Ubuntu 20.04(x64)` recommended. Since multiple versions of Python environments might be present on the system, it is advisable to use miniforge3 to manage Python environments.
```
# Check if miniforge3 and conda are installed, and if they are, you can skip this section.
conda -V
# Download the miniforge3 installer package.
wget -c https://mirrors.bfsu.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
# Install miniforge3
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
# Activate the Conda base environment. The installation directory for miniforge3 will be the default location for Conda.
source ~/miniforge3/bin/activate
# Create a Conda environment named RKLLM-Toolkit with Python 3.8 (recommended version).
conda create -n RKLLM-Toolkit python=3.8
# Enter RKLLM-Toolkit Conda environment
conda activate RKLLM-Toolkit
# Install RKLLM-Toolkit,such as rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
pip3 install rkllm-toolkit/rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
```
If the following command executes without errors, the installation was successful:
```
python
from rkllm.api import RKLLM
```
## Large Language Model Deployment Example

The large language models supported by RKLLM are as follows:

| Model               | Huggingface Link                                                          					|
| ------------------- | ------------------------------------------------------------------------------------------------------ |
| DeepSeek-R1-Distill | [LINK](https://huggingface.co/collections/deepseek-ai/deepseek-r1-678e1e131c0169c0bc89728d)            |
| LLAMA               | [LINK](https://huggingface.co/meta-llama)                                                              |
| TinyLLAMA           | [LINK](https://huggingface.co/TinyLlama)                                                               |
| Qwen                | [LINK](https://huggingface.co/models?search=Qwen/Qwen)                                                 |
| Phi                 | [LINK](https://huggingface.co/models?search=microsoft/phi)                                             |
| ChatGLM3-6B         | [LINK](https://huggingface.co/THUDM/chatglm3-6b/tree/103caa40027ebfd8450289ca2f278eac4ff26405)         |
| Gemma               | [LINK](https://huggingface.co/collections/google/gemma-2-release-667d6600fd5220e7b967f315)             |
| InternLM2           | [LINK](https://huggingface.co/collections/internlm/internlm2-65b0ce04970888799707893c)                 |
| MiniCPM             | [LINK](https://huggingface.co/collections/openbmb/minicpm-65d48bf958302b9fd25b698f)                    |
| TeleChat            | [LINK](https://huggingface.co/Tele-AI)                                                                 |
| Qwen2-VL            | [LINK](https://huggingface.co/Qwen/Qwen2-VL-2B-Instruct)                                               |
| MiniCPM-V           | [LINK](https://huggingface.co/openbmb/MiniCPM-V-2_6)                                                   |

### Large Language Model Conversion And Execution Demo
Below is an example demonstrating how to convert, quantize, export a large language model, and finally deploy and run it on the board, using the `DeepSeek-R1-Distill-Qwen-1.5B_Demo` provided by the RKLLM SDK.
#### Convert The Model On The PC
Using DeepSeek-R1-Distill-Qwen-1.5B as an example, click the table link and find the corresponding repository to clone the [complete repository](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B/tree/main).

With RKLLM-Toolkit properly installed, activate the Conda base environment and execute `generate_data_quant.py`. This will generation results as the quantization calibration data.
```
# Note that you need to set the path to the repository that you cloned.
cd export
python3 generate_data_quant.py -m ~/DeepSeek-R1-Distill-Qwen-1.5B
```
Modify `export_rkllm.py` according to the actual situation. Execute `export_rkllm.py`.You can get the rkllm model.
```
modelpath = '/rkllm/rkllm_model/DeepSeek-R1-Distill-Qwen-1.5B'
# For RK3576, Quantization type is W4A16,and it is a dual-core NPU.Then no change is required for RK3588.
target_platform = "RK3576"
quantized_dtype = "W4A16"
num_npu_core = 2
```

```
(RKLLM-Toolkit-1.1.4) root@ea5d57ca8e66:/rkllm/rknn-llm/examples/DeepSeek-R1-Distill-Qwen-1.5B_Demo/export# python3 export_rkllm.py 
INFO: rkllm-toolkit version: 1.1.4
WARNING: Cuda device not available! switch to cpu device!
The argument `trust_remote_code` is to be used with Auto classes. It has no effect here and is ignored.
Downloading data files: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 15307.68it/s]
Extracting data files: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 2377.72it/s]
Generating train split: 21 examples [00:00, 3327.05 examples/s]
Optimizing model: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 28/28 [21:16<00:00, 45.59s/it]
Building model: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 399/399 [00:07<00:00, 52.78it/s]
WARNING: The bos token has two ids: 151646 and 151643, please ensure that the bos token ids in config.json and tokenizer_config.json are consistent!
INFO: The token_id of bos is set to 151646
INFO: The token_id of eos is set to 151643
INFO: The token_id of pad is set to 151643
Converting model: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 339/339 [00:00<00:00, 2494507.12it/s]
INFO: Exporting the model, please wait ....
[=================================================>] 597/597 (100%)
INFO: Model has been saved to ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm!
```
#### Deploy And Run The RKLLM Model On CQ38W-3576.

##### Kernel Requirements
Before performing model inference with RKLLM Runtime, you must first verify that the NPU kernel on the board is version `v0.9.8` or higher.
```
root@firefly:/# cat /sys/kernel/debug/rknpu/version
RKNPU driver: v0.9.8
```
If the NPU kernel version is lower than v0.9.8, please go to the official firmware address to download the latest firmware
##### Compilation Requirements for RKLLM Runtime
When using RKLLM Runtime, pay attention to the version of the GCC cross-compilation toolchain. It is recommended to use cross-compilation toolchain [gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu](https://developer.arm.com/downloads/-/gnu-a/10-2-2020-11)
```
cd deploy
# Modify build-linux.sh
GCC_COMPILER_PATH=~/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu
# Add executable permissions
chmod +x ./build-linux.sh
# Compilation
./build-linux.sh
```
##### Running Inference On CQ38W-3576
After compiling, generate the executable program `install/demo_Linux_aarch64/llm_demo`. Transfer the compiled `llm_demo` executable and the library file `rkllm-runtime/runtime/Linux/librkllm_api/aarch64/librkllmrt.so` and fixed frequency script `rknn-llm/scripts/fix_freq_rk3588.sh` to the board using `scp` command.
```
firefly@firefly:~$ ./fix_freq_rk3588.sh
firefly@firefly:~$ export LD_LIBRARY_PATH=./lib
firefly@firefly:~$ taskset f0 ./llm_demo ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096
rkllm init start
I rkllm: rkllm-runtime version: 1.1.4, rknpu driver version: 0.9.8, platform: RK3588

rkllm init success

**********************可输入以下问题对应序号获取回答/或自定义输入********************

[0] 现有一笼子，里面有鸡和兔子若干只，数一数，共有头14个，腿38条，求鸡和兔子各有多少只？
[1] 有28位小朋友排成一行,从左边开始数第10位是学豆,从右边开始数他是第几位?

*************************************************************************


user: who are you?
robot: <think>

</think>

Greetings! I'm DeepSeek-R1, an artificial intelligence assistant created by DeepSeek. I'm at your service and would be delighted to assist you with any inquiries or tasks you may have.

user: 
```
## Others
For more demo and api usage, refer to RKLLM SDK routines and documentation

## FAQs

**Q1: Failed to convert model?**

A1: Check the available RAM on your PC.Models with larger parameter sizes require more memory for conversion or execution. You might consider increasing the swapfile size or using a PC with more memory.
