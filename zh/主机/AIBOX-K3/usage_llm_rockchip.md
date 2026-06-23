# 大语言模型
## 1.RKLLM 介绍
RKLLM SDK可以帮助用户快速将大语言模型部署到AIBOX-K3上。
[SDK下载](https://github.com/airockchip/rknn-llm)
### 1.1 RKLLM-Toolkit 功能介绍
RKLLM-Toolkit 是为用户提供在计算机上进行大语言模型的量化、转换的开发套件。通过该工具提供的 Python 接口可以便捷地完成以下功能：
* 模型转换：支持将 Hugging Face 格式的大语言模型（Large Language Model, LLM）转换为RKLLM 模型，转换后的 RKLLM 模型能够在 Rockchip NPU 平台上加载使用。
* 量化功能：支持将浮点模型量化为定点模型，目前支持的量化类型包括 w4a16 和 w8a8。
### 1.2 RKLLM Runtime 功能介绍
RKLLM Runtime 主 要 负 责 加 载 RKLLM-Toolkit 转换得到的 RKLLM 模型，并在AIBOX-K3 板端通过调用 NPU 驱动在 Rockchip NPU 上实现 RKLLM 模型的推理。在推理RKLLM 模型时，用户可以自行定义 RKLLM 模型的推理参数设置，定义不同的文本生成方式，并通过预先定义的回调函数不断获得模型的推理结果。

## 2.RKLLM-Toolkit 安装
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
## 3.大语言模型部署例程
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

### 3.1 大语言模型转换运行Demo
下面以RKLLM SDK的`DeepSeek-R1-Distill-Qwen-1.5B_Demo`为例演示如何转换、量化、导出大语言模型，最后在板端部署运行。
#### 3.1.1 在 PC 上转换模型
DeepSeek-R1-Distill-Qwen-1.5B 为例，依据表格链接，找到对应[模型仓库](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B/tree/main)，克隆`完整仓库`内容。
在前述RKLLM-Toolkit安装正确的前提下，进入 Conda 环境。然后执行`generate_data_quant.py`生成量化标定数据。
```
# 注意设置正确的模型仓库路径
cd export
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
#### 3.1.2 在 AIBOX-K3 上部署运行RKLLM模型
##### 3.1.2.1 板端内核要求
在板端使用RKLLM Runtime 进行模型推理前，首先需要确认板端的NPU 内核是否为`v0.9.8`版本以上
```
root@firefly:/# cat /sys/kernel/debug/rknpu/version
RKNPU driver: v0.9.8
```
若所查询的 NPU 内核版本低于 v0.9.8，请前往官方固件地址下载最新固件进行更新

##### 3.1.2.2 RKLLM Runtime 的编译要求
在使用 RKLLM Runtime 的过程中，需要注意gcc交叉编译工具的版本。推荐下载交叉编译工具[gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu](https://developer.arm.com/downloads/-/gnu-a/10-2-2020-11)
```
cd deploy
# 修改 build-linux.sh，设置交叉编译器路径为本地工具所在路径
GCC_COMPILER_PATH=~/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu
# 添加可执行权限
chmod +x ./build-linux.sh
# 编译
./build-linux.sh
```
##### 3.1.2.3 板端运行推理
编译完成后生成可执行程序`install/demo_Linux_aarch64/llm_demo`，将编译得到的`llm_demo`和库文件`rknn-llm/rkllm-runtime/Linux/librkllm_api/aarch64/librkllmrt.so`以及定频脚本`rknn-llm/scripts/fix_freq_rk3588.sh`一并通过`scp`推送到板端。

```
# 执行定频脚本，以最高频率运行
firefly@firefly:~$ ./fix_freq_rk3588.sh
# 指定函数库路径，运行demo
firefly@firefly:~$ export LD_LIBRARY_PATH=./lib
firefly@firefly:~$ taskset f0 ./llm_demo ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096
rkllm init start
I rkllm: rkllm-runtime version: 1.1.4, rknpu driver version: 0.9.8, platform: RK3588

rkllm init success

**********************可输入以下问题对应序号获取回答/或自定义输入********************

[0] 现有一笼子，里面有鸡和兔子若干只，数一数，共有头14个，腿38条，求鸡和兔子各有多少只？
[1] 有28位小朋友排成一行,从左边开始数第10位是学豆,从右边开始数他是第几位?

*************************************************************************


user: 你是谁？
robot: <think>

</think>

您好！我是由中国的深度求索（DeepSeek）公司开发的智能助手DeepSeek-R1。如您有任何任何问题，我会尽我所能为您提供帮助。

user: 

```
##### 3.1.2.4 聊天模板
LLM 的一个常见应用场景是聊天。在聊天上下文中，不再是连续的文本字符串构成的语句，聊天模型由一条或多条消息组成的对话组成，每条消息都有一个“用户“或“助手”等角色，还包括消息文本。然而不同的模型对聊天的输入格式要求也不同。聊天模板用于将问答的对话内容转换为模型的输入prompt。倘若使用空的聊天模板或者使用默认的聊天模板，与模型在训练期间看到的消息格式不匹配，可能会导致性能下降。

例如查看ChatGLM3的[聊天模板](https://github.com/THUDM/ChatGLM3/tree/main)
```
<|system|>
You are ChatGLM3, a large language model trained by Zhipu.AI. Follow the user's instructions carefully. Respond using markdown.
<|user|>
Hello
<|assistant|>
Hello, I'm ChatGLM3. What can I assist you today?
```
## 4.其他
更多demo和api使用请参考RKLLM SDK例程和文档。

## 5.FAQs

**Q1：转换模型失败？**

A1：检查 PC 可用运行内存，参数量越大的模型转换或者运行时所需的内存越大，可以尝试增大 swapfile 或者使用大内存的 PC。