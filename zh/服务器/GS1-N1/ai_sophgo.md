# SOPHGO 平台
## TPU MLIR
TPU MLIR 是算能深度学处理器的 TPU 编译器工程。该工程提供了一套完整的工具链，其可以将不同框架下预训练的神经网络，转化为可以在算能 TPU 上高效运算的二进制文件 bmodel。

目前该工程直接支持的深度学习框架包括 PyTorch、ONNX、TFLite 和 Caffe，其他框架模型需要转成 ONNX。

也支持编译 HuggingFace LLM 模型，目前支持 qwen2 系列/ llama 系列，后续会支持更多类型的 LLM 模型。

[了解详情](https://github.com/sophgo/tpu-mlir)

## TPU LLM
TPU LLM 实现算能 BM1684X、BM1688(CV186X) 芯片部署各类开源生成式 AI 模型，其中以 LLM/VLM 为主。通过 TPU-MLIR 编译器将模型转换成 bmodel，再基于 tpu-runtime 的推理引擎接口，采用 python/c++ 代码将其部署到 PCIE 环境或者 SoC 环境。

如果要编译模型，需要配置 TPU-MLIR 环境，包括安装 docker 和编译源码； 也可以直接用各类 Demo 中编译好的 bmodel。

[了解详情](https://github.com/sophgo/LLM-TPU)