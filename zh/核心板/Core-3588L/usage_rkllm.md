# 大语言模型
## RKLLM 介绍
RKLLM SDK可以帮助用户快速将大语言模型部署到AIO-3588L上。
[SDK下载](https://github.com/airockchip/rknn-llm)
### RKLLM-Toolkit 功能介绍
RKLLM-Toolkit 是为用户提供在计算机上进行大语言模型的量化、转换的开发套件。通过该工具提供的 Python 接口可以便捷地完成以下功能：
* 模型转换：支持将 Hugging Face 格式的大语言模型（Large Language Model, LLM）转换为RKLLM 模型，转换后的 RKLLM 模型能够在 Rockchip NPU 平台上加载使用。
* 量化功能：支持将浮点模型量化为定点模型，目前支持的量化类型包括 w4a16 和 w8a8。
### RKLLM Runtime 功能介绍
RKLLM Runtime 主 要 负 责 加 载 RKLLM-Toolkit 转换得到的 RKLLM 模型，并在AIO-3588L 板端通过调用 NPU 驱动在 Rockchip NPU 上实现 RKLLM 模型的推理。在推理RKLLM 模型时，用户可以自行定义 RKLLM 模型的推理参数设置，定义不同的文本生成方式，并通过预先定义的回调函数不断获得模型的推理结果。

## RKLLM-Toolkit 安装
RKLLM-Toolkit目前只适用于Linux PC，建议使用`Ubuntu20.04(x64)`。因为系统中可能同时有多个版本的 Python 环境，建议使用 miniforge3 管理 Python 环境。
```
# 检查是否安装 miniforge3 和 conda 版本信息，若已安装则可省略此小节步骤。
conda -V
# 下载 miniforge3 安装包
wget -c https://mirrors.bfsu.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
# 安装 miniforge3
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
# 进入Conda base 环境，miniforge3 为安装目录
source ~/miniforge3/bin/activate
# 创建一个 Python3.8 版本（建议版本）名为 RKLLM-Toolkit 的 Conda 环境
conda create -n RKLLM-Toolkit python=3.8
# 进入 RKLLM-Toolkit Conda 环境
conda activate RKLLM-Toolkit
# 安装 RKLLM-Toolkit，如rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
pip3 install rkllm-toolkit/rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
```
若执行以下命令没有报错，则安装成功。
```
python
from rkllm.api import RKLLM
```
## 大语言模型部署例程
RKLLM已支持的大模型如下

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

### 大语言模型转换运行Demo
下面以RKLLM SDK的`rkllm_api_demo`为例演示如何转换、量化、导出大语言模型，最后在板端部署运行。
#### 在 PC 上转换模型
以DeepSeek-R1-Distill-Qwen-1.5B 模型为例，依据表格链接，找到对应[模型仓库](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B/tree/main)，克隆`完整仓库`内容。
在前述RKLLM-Toolkit安装正确的前提下，进入 Conda 环境。然后执行`generate_data_quant.py`生成量化标定数据。
```
# 注意设置正确的模型仓库路径
cd rknn-llm/examples/rkllm_api_demo/export
python3 generate_data_quant.py -m ~/DeepSeek-R1-Distill-Qwen-1.5B
```
如下按实际情况修改`export_rkllm.py`之后。执行`export_rkllm.py`。即可得到转换模型。
```
# 修改为实际路径
modelpath = '/rkllm/rkllm_model/DeepSeek-R1-Distill-Qwen-1.5B'
# 对于 RK3576， 量化方式为W4A16，且为双核NPU。RK3588则无需修改。
target_platform = "RK3576"
quantized_dtype = "W4A16"
num_npu_core = 2
```

```
lanzj@DL-Station:~/rknn-llm/examples/rkllm_api_demo/export$ python3 export_rkllm.py
INFO: rkllm-toolkit version: 1.2.3
Building model: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 371/371 [00:05<00:00, 69.25it/s]
Generating train split: 21 examples [00:00, 2604.93 examples/s]
Optimizing model: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 28/28 [00:26<00:00,  1.07it/s]
WARNING: The bos token has two ids: 151646 and 151643, please ensure that the bos token ids in config.json and tokenizer_config.json are consistent!
INFO: Setting token_id of bos to 151646
INFO: Setting token_id of eos to 151643
INFO: Setting add_bos_token to True
INFO: Setting add_eos_token to False
Converting model: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 339/339 [00:00<00:00, 4499585.62it/s]
INFO: Setting max_context_limit to 4096
INFO: Exporting the model, please wait ....
[=================================================>] 597/597 (100%)
INFO: Model has been saved to ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm!
```
#### 在 AIO-3588L 上部署运行RKLLM模型
##### 板端内核要求
在板端使用RKLLM Runtime 进行模型推理前，首先需要确认板端的NPU 内核是否为`v0.9.8`版本以上
```
root@firefly:/# cat /sys/kernel/debug/rknpu/version
RKNPU driver: v0.9.8
```
若所查询的 NPU 内核版本低于 v0.9.8，请前往官方固件地址下载最新固件进行更新

##### RKLLM Runtime 的编译要求
在使用 RKLLM Runtime 的过程中，需要注意gcc交叉编译工具的版本。推荐下载交叉编译工具[gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu](https://pan.baidu.com/s/1HOB492BcduU4LFHJM2-Y3w?pwd=1234)。
若是选择使用 Android 平台，需要进行 Android 可执行文件的编译，推荐使用 Android NDK 工具进行交叉编译，下载路径为：[Android_NDK 交叉编译工具下载地址](https://github.com/android/ndk/wiki/Unsupported-Downloads)，推荐使用 r21e 版本。
```
cd rknn-llm/examples/rkllm_api_demo/
# Linux 平台
# 修改 build-linux.sh，设置交叉编译器路径为本地工具所在路径
GCC_COMPILER_PATH=~/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu
# 编译
./build-linux.sh
# Android 平台
# 修改 build-linux.sh，设置交叉编译器路径为本地工具所在路径
ANDROID_NDK_PATH=~/android-ndk-r21e
# 编译
./build-android.sh
```
##### 板端运行推理
编译完成后生成目录:install/demo_Linux_aarch64，将所需文件推送到设备端，若板端不支持adb也可通过scp发送。
```
# 推送目录到设备端
adb push install/demo_Linux_aarch64 /data
# 推送转换好的模型到设备端
adb push rknn-llm/examples/rkllm_api_demo/export/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm /data/demo_Linux_aarch64
# 推送定频脚本到设备端
adb push rknn-llm/scripts/fix_freq_rk3588.sh /data/demo_Linux_aarch64
```
进入设备端运行
```
adb shell
cd /data/demo_Linux_aarch64
# 设置库路径
export LD_LIBRARY_PATH=./lib
# 执行定频脚本，以最高频率运行
sh ./fix_freq_rk3588.sh
# 设置log等级
export RKLLM_LOG_LEVEL=1
# 运行demo
./llm_demo ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096

# Running result                                                          
rkllm init start
rkllm init success

**********************可输入以下问题对应序号获取回答/或自定义输入********************

[0] 现有一笼子，里面有鸡和兔子若干只，数一数，共有头14个，腿38条，求鸡和兔子各有多少只？
[1] 有28位小朋友排成一行,从左边开始数第10位是学豆,从右边开始数他是第几位?

*************************************************************************

user: 
```
## 其他
更多demo和api使用请参考RKLLM SDK例程和文档。

## FAQs

**Q1：转换模型失败？**

A1：检查 PC 可用运行内存，参数量越大的模型转换或者运行时所需的内存越大，可以尝试增大 swapfile 或者使用大内存的 PC。
