# AI 教程

## AidLux 介绍

AidLux 是一套完备的边缘端 AI 开发工具套件，能简化高通平台的环境部署，帮助开发者加速 AI 应用落地。

我们推荐使用 AidLux 来进行 AI 应用的开发。AidLux 包含 AidLite、AidGen、AidGenSE、AidStream 等组件。

## 系统依赖配置

* 配置 AidLux 源

```bash
# 下载正确的公钥
sudo wget -O- https://archive.aidlux.com/ubuntu24/public.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/private-aidlux.gpg > /dev/null

# 编辑源文件
sudo vim /etc/apt/sources.list.d/private-aidlux.list

# 在源文件中填入AidLux 提供的私钥
deb [arch=arm64 signed-by=/etc/apt/trusted.gpg.d/private-aidlux.gpg] https://archive.aidlux.com/ubuntu24 noble main

# 更新缓存
sudo apt update
```

* 安装 AidLux 依赖
```bash
# 必须先安装的，系统不自带
sudo apt install python3 python3-pip libopencv-dev python3-opencv  net-tools

# 安装依赖
sudo apt install aidlux-aistack-base aidrtcm
sudo apt install aid-lms aidlms-sdk cmake aid-mms

# DSP 支持
sudo apt-get install qcom-fastrpc1
sudo apt-get install qcom-fastrpc-dev

# GPU 支持
sudo apt-add-repository -s ppa:ubuntu-qcom-iot/qcom-ppa
sudo apt install qcom-adreno-cl1
sudo ln -s /usr/lib/aarch64-linux-gnu/libOpenCL.so.1 /usr/lib/aarch64-linux-gnu/libOpenCL.so

# 支持 aidlite-sdk 和 aidgen-sdk
sudo apt install aidlite-sdk aidlite-*
sudo apt install aidgen-qnn240-sdk
sudo apt-get install libfmt-dev nlohmann-json3-dev
```

安装完成后，检查系统 /usr/local/share/ 新增 aidlite 和 aidgen 目录。

![](../../../qcom_img/AIBOX-9075/check_aid_files.png)

## 模型广场

模型广场聚集了大量的经过优化适配在高通 NPU 上的前沿 AI 模型，结合 AidLux 工具链，可以帮助开发者在高通芯片上更快的构建自己的 AI 应用。

地址：https://aiot.aidlux.com/zh/models

请前往注册账号，后续的例子会用到。

## AidLite

AidLite 是 AI 执行框架，旨在充分调度端侧芯片的各计算单元 (CPU、GPU、NPU) 实现AI模型的加速推理。

例子：

### YOLOv8s

YOLOv8 是一款前沿的、最先进的（SOTA）模型，基于之前YOLO版本的成功进行了构建，并引入了新功能和改进，进一步提升了性能和灵活性。YOLOv8 设计快速、准确且易于使用，是广泛应用于物体检测与跟踪、实例分割、图像分类以及姿态估计任务的优秀选择。

* 通过 mms 命令下载对应的 yolov8s 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep YOLOv8s | grep FP16
mms get -m YOLOv8s -p fp16 -c iq9 -b qnn2.36 -d ./

# 解压
unzip YOLOv8s_qcs8550_fp16.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
cd ./code
python3 python/run_test.py --target_model ../models/IQ9/FP16/yolov8s_qcs9100_fp16.qnn236.ctx.bin --imgs ./python/bus.jpg  --invoke_nums 10
```

![](../../../qcom_img/AIBOX-9075/bus_input.jpg)

![](../../../qcom_img/AIBOX-9075/yolov8s_run.jpg)

* 检查生成的结果 python/result.jpg

![](../../../qcom_img/AIBOX-9075/yolov8s_result.jpg)

### ControlNet

ControlNet 是一种用于增强生成模型控制能力的神经网络框架，通过引入额外条件输入（如边缘检测、深度图、语义分割等）实现精细化图像生成。该模型由 Lvmin Zhang 和 Maneesh Agrawala 等人提出，可无缝集成到扩散模型中，允许用户通过草图、姿态或结构约束精准控制生成内容的构图、物体位置及细节风格，广泛应用于艺术创作、设计辅助、影视预可视化及图像编辑。其核心在于条件编码器与预训练模型的联合训练，平衡生成自由度与控制强度，支持多模态条件输入与实时交互。技术挑战包括复杂条件融合的稳定性、计算效率优化及过拟合控制。

* 通过mms 命令下载对应的ControlNet 模型，这里需要注册Aplux  模型广场的账户

```bash
#登录,根据提示输入模型广场的账户和密码
mms login
#列举并下载模型
mms list | grep ControlNet | grep FP16
mms get -m ControlNet -p w8a16 -c qcs8550 -b qnn2.36 -d ./
unzip controlnet_qcs8550_qnn2.36_w8a16_aidlite.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
pip install transformers==5.1.0 diffusers==0.36.0
cd model_farm_controlnet_qcs8550_qnn2.36_w8a16_aidlite
python3 python/run_test.py
```
![](../../../qcom_img/AIBOX-9075/controlnet_input.jpg)

![](../../../qcom_img/AIBOX-9075/controlnet_run.jpg)

* 检查生成的结果 python/output.jpg

![](../../../qcom_img/AIBOX-9075/controlnet_result.jpg)

### ConvNeXt-Tiny

ConvNeXt-Tiny是ConvNeXt模型家族中的轻量级版本，它是一种现代卷积神经网络（CNN），旨在对传统CNN进行重新设计，以与当前流行的Transformer模型相竞争。ConvNeXt-Tiny保留了卷积网络的优势，同时引入了许多来自视觉Transformer的设计思想，如更深的网络结构、更大的卷积核和LayerNorm等。与其他模型相比，ConvNeXt-Tiny具有更少的参数和计算需求，但仍能提供高效的图像分类性能，因此特别适合在资源有限的环境中使用，如移动设备或边缘计算。

* 通过mms 命令下载对应的ConvNeXt-Tiny 模型，这里需要注册Aplux  模型广场的账户

```bash
#登录,根据提示输入模型广场的账户和密码
mms login
#列举并下载模型
mms list | grep ConvNeXt-Tiny | grep FP16 | grep 8550
mms get -m ConvNeXt-Tiny -p FP16 -c QCS8550 -b qnn2.36 -d ./
unzip ConvNeXt-Tiny_qcs8550_fp16.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
cd ./code
python3  python/run_test.py --target_model ../models/QCS8550/FP16/convnext_tiny_qcs8550_fp16.qnn236.ctx.bin --imgs ./python/tiger_cat.jpg  --invoke_nums 10
```
![](../../../qcom_img/AIBOX-9075/convnext-tiny_input.jpg)

![](../../../qcom_img/AIBOX-9075/convnext-tiny_run.jpg)


### Depth-Anything-V2-Small

Depth-Anything-V2 是单目深度估计领域的先进基础模型，旨在从单张 RGB 图像中预测高精度的深度信息。相比于第一代模型，V2 版本通过采用更强大的教师模型（基于 DINOv2-Giant）以及更大规模的合成数据（59.5万张）与伪标签真实数据（6200万张）联合训练，显著提升了对复杂场景和微小细节的捕捉能力。它不仅在处理透明物体、高反射表面及弱光环境时更具鲁棒性，还通过优化的推理架构实现了比同类扩散模型快 10 倍以上的速度。Depth-Anything-V2 提供了从轻量级（25M）到超大规模（1.3B）的多种尺寸，能够完美适配从嵌入式实时应用到高精度 3D 重建的多元化需求。

* 通过mms 命令下载对应的Depth-Anything-V2-Small 模型，这里需要注册Aplux  模型广场的账户

```bash
#登录,根据提示输入模型广场的账户和密码
mms login
#列举并下载模型
mms list | grep Depth-Anything-V2-Small | grep FP16 | grep 8550
mms get -m Depth-Anything-V2-Small -p FP16 -c iq9 -b qnn2.36 -d ./
unzip Depth-Anything-V2-Small_iq9_fp16.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
pip install  Pillow numpy matplotlib
cd code
cp python/* ./
python3 run_test.py
```
![](../../../qcom_img/AIBOX-9075/depth_input.jpg)

![](../../../qcom_img/AIBOX-9075/depth_run.jpg)

* 检查生成的结果

![](../../../qcom_img/AIBOX-9075/depth_result.png)

### FastSAM-S

Fast Segment Anything Model (FastSAM) 是一种新颖的、基于CNN的实时解决方案，专为“Segment Anything”任务设计。该任务旨在根据多种可能的用户交互提示对图像中的任何物体进行分割。FastSAM 大幅降低了计算需求，同时保持了具有竞争力的性能，使其成为各种视觉任务中的实用选择。

* 通过mms 命令下载对应的FastSAM-S 模型，这里需要注册Aplux  模型广场的账户

```bash
#登录,根据提示输入模型广场的账户和密码
mms login
#列举并下载模型
mms list | grep FastSAM-S | grep FP16
mms get -m FastSAM-S -p FP16 -c iq9 -b qnn2.36 -d ./
unzip FastSAM-S_iq9_fp16.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
cd code
python3 python/run_test.py --target_model ../models/IQ9/FP16/cutoff_fastsam_s_qcs9100_fp16.qnn236.ctx.bin --imgs python/dogs.jpg --invoke_nums 10
```
![](../../../qcom_img/AIBOX-9075/fastsam_input.jpg)

![](../../../qcom_img/AIBOX-9075/fastsam_run.jpg)

* 检查生成的结果

![](../../../qcom_img/AIBOX-9075/fastsam_result.jpg)

### YOLO11l-Pose

YOLO11 pose 是 Ultralytics 姿态估计技术的最新巅峰，涵盖了从极致轻量化（Nano）到高精度（Extra-Large）的完整模型阵列。该系列采用了全新的主干网络与颈部架构设计，通过引入改进的 C3k2 模块和 C2PSA 机制，在精度与推理速度的帕累托前沿上取得了显著突破。相比前代版本，它能更精准地捕捉复杂人体姿态，并在严苛的遮挡环境下保持关键点追踪的稳定性。无论是对实时性要求极高的边缘侧端计算，还是追求最高学术指标的离线分析任务，YOLO11 pose 都能通过其灵活的可伸缩性提供最优的 2D 关键点检测方案。

* 通过mms 命令下载对应的YOLO11l-Pose 模型，这里需要注册Aplux  模型广场的账户

```bash
#登录,根据提示输入模型广场的账户和密码
mms login
#列举并下载模型
mms list | grep YOLO11l-Pose | grep FP16
mms get -m YOLO11l-Pose -p FP16 -c iq9 -b qnn2.36 -d ./
unzip YOLO11l-Pose_iq9_fp16.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
cd code
python3 python/run_test.py --target_model ../models/IQ9/FP16/yolo11l-pose_qcs9100_fp16.qnn236.ctx.bin --imgs python/bus.jpg --invoke_nums 10
```
![](../../../qcom_img/AIBOX-9075/yolo11l-pose_input.jpg)

![](../../../qcom_img/AIBOX-9075/yolo11l-pose_run.jpg)

* 检查生成的结果

![](../../../qcom_img/AIBOX-9075/yolo11l-pose_result.jpg)

### YOLO11s-obb
**YOLO11s-obb** 是 YOLO 系列中轻量级的定向边界框（Oriented Bounding Box，OBB）检测模型，优化了传统 YOLO 架构以支持旋转物体的高效识别，适合资源受限的嵌入式和移动端应用。

**主要特性**：

- **Anchor-Free 结构**：采用 anchor-free 设计，简化模型并提升推理速度；
- **方向检测头**：除了位置和类别，该模型还能预测目标的旋转角度，实现对旋转物体（如无人机、车辆、路牌、文本区域）的准确定位；
- **轻量化骨干**：结合轻量级骨干网络与定向检测模块，参数量和计算量低，满足近实时推理需求；
- **场景适应性强**：适用于航拍视频分析、工业零件检测、地图元素识别等方向敏感的视觉任务。

* 通过mms 命令下载对应的YOLO11s-obb 模型，这里需要注册Aplux  模型广场的账户

```bash
#登录,根据提示输入模型广场的账户和密码
mms login
#列举并下载模型
mms list | grep YOLO11s-obb | grep FP16 | grep 8550
mms get -m YOLO11s-obb -p FP16 -c QCS8550 -b qnn2.36 -d ./
unzip YOLO11s-obb_qcs8550_fp16.zip
```

* 移动到模型目录，阅读 README 文档中操作执行模型调用
```bash
cd code
sudo python3 python/run_test.py --target_model ../models/QCS8550/FP16/yolo11s-obb_qcs8550_fp16.qnn236.ctx.bin --imgs python/boats.jpg --invoke_nums 10
```
![](../../../qcom_img/AIBOX-9075/yolo11s-obb_input.jpg)

![](../../../qcom_img/AIBOX-9075/yolo11s-obb_run.jpg)

* 检查生成的结果

![](../../../qcom_img/AIBOX-9075/yolo11s-obb_result.jpg)

## AidGen

AidGen 是基于 AidLite 构建的专门针对生成式 Transformer 模型的推理框架，旨在充分调用硬件的各计算单元（CPU、GPU、NPU）实现大模型在端侧的推理加速。

AidGen 提供原子级别的大模型推理接口，适用于开发者将大模型推理集成到自己的应用中。

AidGen 支持多种类型的生成式 AI 模型:

语言类大模型 -> AidLLM 推理

多模态大模型 -> AidMLM 推理

例子：

### MiniCPM5-1B

MiniCPM5-1B 是 MiniCPM5 系列的首个模型，面向本地助手、coding agent、工具调用流程以及需要紧凑模型的推理场景。它在较小部署成本下提供原生长上下文能力，并通过同一份权重支持 Think / No Think 两种对话模式。

* 通过 mms 命令下载对应的 MiniCPM5 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep MiniCPM5
mms get -m MiniCPM5-1B -p W4A16 -c iq9 -b qnn2.36 -d ./
```

* 解压模型
```bash
mkdir MiniCPM5-1B-aidllm
unzip qnn236_qcs9075_cl4096.zip -d MiniCPM5-1B-aidllm/
```

* 新增模型配置文件 config.json
```bash
cd  ./MiniCPM5-1B-aidllm/qnn236_qcs9075_cl4096
vim config.json
```

配置文件内容：
```json
{
    "backend_type" : "genie",
    "prefix_path":"kv-cache.primary.qnn-htp",
    "model":{
        "path" : [
        "minicpm5-1b_qnn236_qcs9075_cl4096_1_of_3.aidem",
	"minicpm5-1b_qnn236_qcs9075_cl4096_2_of_3.aidem",
	"minicpm5-1b_qnn236_qcs9075_cl4096_3_of_3.aidem"
        ]
    }
}
```

* 拷贝示例工程目录到模型目录并编译
```bash
# 拷贝工程目录到模型目录：
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 编译
cd ./aidllm
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm abort ./config.json
```

![](../../../qcom_img/AIBOX-9075/minicpm5-1b_run.jpg)

### Qwen3-8B-CL8192

Qwen3 是 Qwen 系列最新一代的大语言模型，提供了完整的稠密模型和专家混合（MoE）模型套件。基于大规模训练，Qwen3 在推理、指令跟随、智能体能力以及多语言支持方面实现了突破性进展，主要特性如下：

1. 独特支持在单一模型中无缝切换：可在 思考模式（用于复杂逻辑推理、数学和编程）与 非思考模式（用于高效的通用对话）之间切换，从而在不同场景下都能发挥最佳性能。
2. 推理能力显著增强：在数学、代码生成和常识逻辑推理方面，超越了此前的 QwQ（思考模式）和 Qwen2.5 instruct 模型（非思考模式）。
3. 更优的人类偏好对齐：在创意写作、角色扮演、多轮对话和指令跟随方面表现突出，能够带来更自然、更有吸引力和更沉浸式的对话体验。
4. 智能体能力方面的专长：能够在思考和非思考模式下精确调用外部工具，在复杂的基于智能体任务中实现了开源模型的领先表现。
5. 支持 100+ 种语言和方言，并具备强大的 多语言指令跟随 和 翻译 能力。

* 通过 mms 命令下载对应的 Qwen3 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep Qwen3
mms get -m Qwen3-8B-CL8192 -p W4A16 -c iq9 -b qnn2.36 -d ./
```

* 解压模型
```bash
mkdir qwen3-8b-aidllm
unzip qnn236_qcs9075_cl8192.zip -d qwen3-8b-aidllm/
```

* 新增模型配置文件 qwen3-8b-encrypt.json
```bash
cd  ./qwen3-8b-aidllm/qnn236_qcs9075_cl8192
vim qwen3-8b-encrypt.json
```

配置文件内容：
```json
{
    "backend_type" : "genie",
    "prefix_path":"kv-cache.primary.qnn-htp",
    "model":{
        "path" : [
        "qwen3-8b_qnn236_qcs9075_cl8192_1_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs9075_cl8192_2_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs9075_cl8192_3_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs9075_cl8192_4_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs9075_cl8192_5_of_5.serialized.bin.aidem"
        ]
    }
}
```

* 拷贝示例工程目录到模型目录并编译
```bash
# 拷贝工程目录到模型目录：
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 编译
cd ./aidllm
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm abort ./qwen3-8b-encrypt.json
```

![](../../../qcom_img/AIBOX-9075/qwen3-8b_run.jpg)

### Meta-Llama-3.1-8B-Instruct

Meta Llama 3.1系列是一个多语言大规模语言模型（LLM）集合，包含了8B、70B和405B尺寸的预训练和指令调优生成模型（文本输入/文本输出）。Llama 3.1指令调优的文本模型（8B、70B、405B）针对多语言对话场景进行了优化，在常见的行业基准测试中超过了许多现有的开源和闭源聊天模型。

* 通过 mms 命令下载对应的 Meta-Llama 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep Meta-Llama
mms get -m Meta-Llama-3.1-8B-Instruct -p W4A16 -c iq9 -b qnn2.40 -d ./
```

* 解压模型
```bash
mkdir Meta-Llama-3.1-8B-aidllm
unzip qnn240_qcs9075_cl4096.zip -d Meta-Llama-3.1-8B-aidllm
```

* 拷贝示例工程目录到模型目录，并修改模板
```bash
# 拷贝工程目录到模型目录：
cd Meta-Llama-3.1-8B-aidllm/qnn240_qcs9075_cl4096
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 修改模板
cd ./aidllm
vim test_aidllm.cpp

# 将第 48 行的 prompt_template 修改为如下：
prompt_template = "<|begin_of_text|><|start_header_id|>system<|end_header_id|>\nYou are a pirate chatbot who always responds in pirate speak!<|eot_id|><|start_header_id|>user<|end_header_id|>\n{0}<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n";

# 编译
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm multi_turn ./Meta-Llama-3.1-8B-Instruct-htp.json
```

![](../../../qcom_img/AIBOX-9075/meta-llama-8b_run.jpg)

### Gemma-2-2B-it

Gemma 是 Google 推出的一系列轻量级、最先进的开源模型，它们基于与 Gemini 模型相同的研究成果和技术构建而成。Gemma 属于文本到文本的解码器结构大语言模型（decoder-only LLM），目前支持英文，并提供了预训练版本和指令微调版本的开源权重。Gemma 模型非常适用于多种文本生成任务，包括：问答、摘要生成、推理等。由于模型体积较小，Gemma 可以部署在资源有限的环境中，例如笔记本电脑、台式机或个人云服务，从而让更多人能够接触并使用先进的 AI 模型，推动全民创新。

* 通过 mms 命令下载对应的 Gemma 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep Gemma
mms get -m Gemma-2-2B-it -p W4A16 -c qcs8550 -b qnn2.29 -d ./
```

* 解压模型
```bash
mkdir -p Gemma-2-2B-aidllm
unzip qnn229_qcs8550_cl4096.zip -d Gemma-2-2B-aidllm/
```

* 拷贝示例工程目录到模型目录，并修改模板
```bash
# 拷贝工程目录到模型目录：
cd Gemma-2-2B-aidllm/qnn229_qcs8550_cl4096
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 修改模板
cd ./aidllm
vim test_aidllm.cpp

# 将第 48 行的 prompt_template 修改为如下：
prompt_template = "<bos><start_of_turn>user\n{0}<end_of_turn>\n<start_of_turn>model";

# 编译
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm abort ./gemma-2-2b-it-htp.json
```

![](../../../qcom_img/AIBOX-9075/gemma-2-2b_run.jpg)

### Falcon3-7B-Instruct

Falcon3 开源基础模型系列是一组从 1B 到 10B 参数规模的预训练与指令微调大语言模型。

本仓库包含 Falcon3-7B-Instruct 模型。该模型在推理、语言理解、指令跟随、代码和数学任务上（在发布时）达到了业界最先进的效果。

Falcon3-7B-Instruct 支持 4 种语言（英语、法语、西班牙语和葡萄牙语），并支持最长 32K 的上下文窗口。

* 通过 mms 命令下载对应的 Falcon 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep Falcon
mms get -m Falcon3-7B-Instruct -p W4A16 -c qcs8550 -b qnn2.38 -d ./
```

* 解压模型
```bash
mkdir -p Falcon3-7B-aidllm
unzip qnn237_qcs8550_cl4096.zip -d Falcon3-7B-aidllm/
```

* 拷贝示例工程目录到模型目录，并修改模板
```bash
# 拷贝工程目录到模型目录：
cd Falcon3-7B-aidllm/qnn237_qcs8550_cl4096
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 修改模板
cd ./aidllm
vim test_aidllm.cpp

# 将第 48 行的 prompt_template 修改为如下：
prompt_template = "<|system|>\nYou are a helpful friendly assistant Falcon3 from TII, try to follow instructions as much as possible.\n<|user|>\n{0}\n<|assistant|>\n";

# 编译
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm abort ./falcon3-7b-instruct-htp.json
```

![](../../../qcom_img/AIBOX-9075/falcon3-7b_run.jpg)

### DeepSeek-R1-Distill-Qwen-7B

为了解决DeepSeek-R1-Zero的问题并进一步提升推理性能，DeepSeek推出了DeepSeek-R1，这款模型在强化学习之前加入了冷启动数据。 DeepSeek-R1在数学、代码和推理任务上达到了与OpenAI-o1相当的性能。 为了支持研究社区，DeepSeek已经开源了DeepSeek-R1-Zero、DeepSeek-R1以及基于Llama和Qwen提炼的六个密集模型。DeepSeek-R1-Distill-Qwen-32B在多个基准测试中优于OpenAI-o1-mini，创造了新的密集模型领域的最新成果。

* 通过 mms 命令下载对应的 DeepSeek 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep DeepSeek-R1
mms get -m DeepSeek-R1-Distill-Qwen-7B -p W4A16 -c QCS8550 -b qnn2.29 -d ./
```

* 解压模型
```bash
mkdir -p DeepSeek-R1-Distill-Qwen-7B-aidllm
unzip qnn229_qcs8550_cl4096.zip -d DeepSeek-R1-Distill-Qwen-7B-aidllm
```

* 拷贝示例工程目录到模型目录，并修改模板
```bash
# 拷贝工程目录到模型目录：
cd DeepSeek-R1-Distill-Qwen-7B-aidllm/qnn229_qcs8550_cl4096
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 修改模板
cd ./aidllm
vim test_aidllm.cpp

# 将第 48 行的 prompt_template 修改为如下：
prompt_template = "<｜begin▁of▁sentence｜>You are Deepseek-R1, an AI assistant created exclusively by the Chinese Company DeepSeek. You'll provide helpful, harmless, and detailed responses to all user inquiries.<｜User｜>Introduce Qualcomm in 100 words<｜Assistant｜>"

# 编译
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm multi_turn DeepSeek-R1-Distill-Qwen-7B-htp.json
```

![](../../../qcom_img/AIBOX-9075/deepseek-r1-distill-qwen-7b_run.jpg)

### Phi-3.5-mini-instruct

Phi-3.5-mini 是一款轻量级、最先进的开源模型，构建基础源自 Phi-3 所使用的数据集，包括合成数据和经过筛选的公开网站内容，重点强调高质量、密集推理的数据。该模型属于 Phi-3 模型家族，支持 128K tokens 的上下文长度。模型经过严格的能力增强流程，结合了监督微调（SFT）、近端策略优化（PPO）以及直接偏好优化（DPO），以确保其对指令的精准执行与稳健的安全性控制。

* 通过 mms 命令下载对应的 Phi 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep Phi-3.5
mms get -m Phi-3.5-mini-instruct -p W4A16 -c QCS8550 -b qnn2.29 -d ./
```

* 解压模型
```bash
mkdir -p Phi-3.5-mini-aidllm
unzip qnn229_qcs8550_cl4096.zip -d Phi-3.5-mini-aidllm
```

* 拷贝示例工程目录到模型目录，并修改模板
```bash
# 拷贝工程目录到模型目录：
cd Phi-3.5-mini-aidllm/qnn229_qcs8550_cl4096
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# 修改模板
cd ./aidllm
vim test_aidllm.cpp

# 将第 48 行的 prompt_template 修改为如下：
prompt_template = "<|system|>\nYou are a helpful assistant.<|end|>\n<|user|>\nHow to explain Internet for a medieval knight?<|end|>\n<|assistant|>\n"

# 编译
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm multi_turn Phi-3.5-mini-instruct-htp.json
```

![](../../../qcom_img/AIBOX-9075/phi-3.5-mini_run.jpg)

### HY-MT1.5-1.8B

混元翻译模型 1.5 版本包含 1.8B 翻译模型 HY-MT1.5-1.8B 和 7B 翻译模型 HY-MT1.5-7B。两款模型均致力于支持 33 种语言的互译，并融入了 5 种民族语言及方言。其中，HY-MT1.5-7B 是 WMT25 冠军模型的升级版，针对解释性翻译和混合语言场景进行了优化，并新增了对术语干预、上下文翻译以及格式化翻译的支持。尽管 HY-MT1.5-1.8B 的参数量不足 HY-MT1.5-7B 的三分之一，但其翻译性能可与较大模型相媲美，兼顾了高速度与高质量。经过量化后，1.8B 模型可部署在边缘设备上并支持实时翻译场景，具有广泛的应用前景。

* 通过 mms 命令下载对应的 HY-MT 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep HY-MT
mms get -m HY-MT1.5-1.8B -p w4a16 -c iq9 -b qnn2.36 -d ./
```

* 解压模型
```bash
mkdir HY-MT1.5-1.8B-aidllm
unzip qnn236_qcs9075_cl2048.zip -d HY-MT1.5-1.8B-aidllm
```

* 新增模型配置文件 config.json
```bash
cd HY-MT1.5-1.8B-aidllm/qnn236_qcs9075_cl2048/
vim config.json
```

配置文件内容：
```json
{
    "backend_type" : "genie",
    "prefix_path":"kv-cache.primary.qnn-htp",
    "model":{
        "path" : [
        "hy-mt1.5-1.8b_qnn236_qcs9075_cl2048_1_of_2.serialized.bin.aidem",
        "hy-mt1.5-1.8b_qnn236_qcs9075_cl2048_2_of_2.serialized.bin.aidem"
        ]
    }
}
```

* 拷贝示例工程目录到模型目录，修改模板并编译
```bash
# 拷贝工程目录到模型目录：
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./


# 修改模板
cd ./aidllm
vim test_aidllm.cpp

# 将第 48 行的 prompt_template 修改为如下：
prompt_template = "<｜hy_begin▁of▁sentence｜><｜hy_place▁holder▁no▁3｜>\n<｜hy_begin▁of▁sentence｜>\n<｜hy_User｜>Translate the following into Chinese, without additional explanation.\n\n{0}\n<｜hy_Assistant｜>\n";

# 编译
mkdir build && cd build
cmake ..
make
```

* 调用执行
```bash
cd ../../
./aidllm/build/test_aidllm multi_turn ./config.json
```

![](../../../qcom_img/AIBOX-9075/hy-mt1.5-1.8b_run.jpg)

### Qwen2.5-VL-3B-Instruct

Qwen2.5-VL 是 Qwen2-VL 视觉-语言模型的增强型。主要增强功能：

1. 视觉理解能力：Qwen2.5-VL 不仅擅长识别花卉、鸟类、鱼类、昆虫等常见物体，还能高效分析图像中的文本、图表、图标、图形及布局。
2. 具备主动性：Qwen2.5-VL 可直接作为视觉代理进行推理和动态操作工具，支持电脑及手机操作。
3. 理解长视频并捕捉事件：Qwen2.5-VL 可理解超过 1 小时的视频，并新增了通过定位相关视频片段捕捉事件的能力。
4. 多格式视觉定位能力：Qwen2.5-VL 能够通过生成边界框或关键点精确定位图像中的物体，并可稳定输出坐标及属性的 JSON 格式数据。
5. 生成结构化输出：针对发票、表单、表格等扫描数据，Qwen2.5-VL 支持结构化内容输出，有助于在金融、商业等场景的应用。

* 通过 mms 命令下载对应的 Qwen3 模型，需要提前注册模型广场的账号。
```bash
# 登录，根据提示输入模型广场的账户和密码
mms login

# 列举并下载模型
mms list | grep Qwen2 | grep VL
mms get -m 'Qwen2.5-VL-3B-Instruct (392x392)' -p w4a16 -c qcs8550 -b qnn2.36 -d ./
```

* 解压模型
```bash
mkdir qwen2.5-vl-aidmlm
unzip qnn236_qcs8550_cl2048.zip -d ./qwen2.5-vl-aidmlm/
```

* 新增配置文件 Qwen2.5-VL-3B-392x392-8550.json
```bash
cd ./qwen2.5-vl-aidmlm/qnn236_qcs8550_cl2048/
vim Qwen2.5-VL-3B-392x392-8550.json
```

配置文件内容：
```json
{
    "vision_model_path": "./veg.serialized.bin.aidem",
    "pos_embed_cos_path": "./position_ids_cos.raw",
    "pos_embed_sin_path": "./position_ids_sin.raw",
    "vocab_embed_path": "./embedding_weights_151936x2048.raw",
    "window_attention_mask_path": "./window_attention_mask.raw",
    "full_attention_mask_path": "./full_attention_mask.raw",
    "llm_path_list": [
        "./qwen2p5-vl-3b_qnn236_qcs8550_cl2048_1_of_1.serialized.bin.aidem"
    ]
}
```

* 拷贝示例工程目录到模型目录并编译
```bash
# 拷贝工程目录到模型目录下
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidmlm/* ./

# 编译可执行程序
mkdir build && cd build
cmake ..
make
```

* 运行
```bash
cd ..
./build/test_aidmlm single qwen25vl3b392 Qwen2.5-VL-3B-392x392-8550.json ~/bus_input.jpg "Describe the scene in the picture"
```

![](../../../qcom_img/AIBOX-9075/bus_input.jpg)

![](../../../qcom_img/AIBOX-9075/qwen2.5-vl-3b_run.jpg)

## AidGenSE

AidGenSE 是基于 AidGen 封装的适配了 OpenAI HTTP 协议的生成式 AI HTTP 服务。开发者可以通过 HTTP 方式调用生成式 AI 并快速集成到自己的应用中。

下面介绍 AidGenSE 的用法：

* 环境准备
```bash
# 配置虚拟运行环境
sudo apt install -y python3-pip python3-venv > /dev/null 2>&1
sudo python3 -m venv /opt/aidlux/aid-python3

# 创建 aid-python3 命令
echo '#!/bin/bash
exec /opt/aidlux/aid-python3/bin/python3 "$@"' | sudo tee /usr/bin/aid-python3 > /dev/null
sudo chmod +x /usr/bin/aid-python3

# 创建 aid-pip3 命令
echo '#!/bin/bash
exec /opt/aidlux/aid-python3/bin/python3 -m pip "$@"' | sudo tee /usr/bin/aid-pip3 > /dev/null
sudo chmod +x /usr/bin/aid-pip3

# 安装 AidGenSE deb
sudo apt install aidgense
sudo aidllm system --sys linux --soc 8550
sudo apt install aid-pkg
```

* 常用指令
```bash
# 查询服务器可供下载的模型
sudo aidllm remote-list api
# 拉取模型
sudo aidllm pull api aplux/Qwen3-8B-8550-cl2048
# 枚举本地已下载的模型
sudo aidllm list api
# 启动运行本地模型
sudo aidllm start api
# 停止运行本地模型
sudo aidllm stop api
```

* start api 之后，即可通过 <设备ip>:8888 使用 OpenAI HTTP 请求进行交互

![](../../../qcom_img/AIBOX-9075/aidllm_start_api.png)

## AidStream

AidStream 是一个基于 GStreamer 的多媒体数据实时处理工具包，适用于构建基于 AI 的视频、图像分析应用和服务。

Demo 演示：

* 安装依赖
```bash
sudo apt-add-repository -s ppa:ubuntu-qcom-iot/qcom-ppa
sudo apt update
sudo apt install weston-autostart

sudo apt install gstreamer1.0-qcom-sample-apps
sudo reboot # 需要重启一次

sudo apt-get install libgstreamer1.0-dev gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-alsa gstreamer1.0-gtk3

sudo apt-get install -y libjsoncpp-dev

sudo ln -s /usr/lib/aarch64-linux-gnu/libjsoncpp.so /usr/lib/aarch64-linux-gnu/libjsoncpp.so.1

sudo apt install gstreamer1.0-rtsp

sudo apt install aidstream-gst
```

* 配置视频输入、输出地址
```bash
cd /usr/local/share/aidstream-gst/conf
# 修改输入和输出地址为当前设备可访问的视频流服务器IP
vim aidstream-gst.conf
```

![](../../../qcom_img/AIBOX-9075/aidstream_conf.png)

* 演示
```bash
mkdir aid-streamer-gst
cd aid-streamer-gst
cp -r /usr/local/share/aidstream-gst/example/ ./
cd example/python
sudo python3 example.py
```

然后通过视频输出地址查看处理结果。