# SOPHGO 
## TPU MLIR
TPU-MLIR is an open-source machine-learning compiler based on MLIR for TPU. This project provides a complete toolchain, which can convert pre-trained neural networks from different frameworks into binary files bmodel that can be efficiently operated on TPUs.

Currently, supported Deep Learning frameworks are PyTorch, ONNX, TFLite and Caffe. Models from other frameworks need to be converted to ONNX models.

It also supports compiling HuggingFace LLM models. Currently, the qwen2 and llama series are supported, with more types of LLM models to be added in the future.

[Learn More](https://github.com/sophgo/tpu-mlir)

## TPU LLM
TPU LLM enables the deployment of various open-source generative AI models on Sophgo BM1684X and BM1688 (CV186X) chips, focusing primarily on LLM/VLM models. The TPU-MLIR compiler converts these models into the bmodel format. Subsequently, leveraging the inference engine interfaces of tpu-runtime, deployment is implemented using Python/C++ code for PCIe or SoC environments.

To compile models, the TPU-MLIR environment must be configured, including Docker installation and source code compilation. Alternatively, precompiled bmodel files from available demos may be used directly.

[Learn More](https://github.com/sophgo/LLM-TPU)