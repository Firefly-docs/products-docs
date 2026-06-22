## 飞桨 FastDeploy

### 介绍
FastDeploy 是一款全场景、易用灵活、极致高效的 AI 推理部署工具。提供开箱即用的云边端部署体验, 支持超过 150+ Text, Vision, Speech和跨模态模型，并实现端到端的推理性能优化。包括图像分类、物体检测、图像分割、人脸检测、人脸识别、关键点检测、抠图、OCR、NLP、TTS等任务，满足开发者多场景、多硬件、多平台的产业部署需求。

目前 FastDeploy 初步支持 rknpu2，可以在 RK3588 平台上推理部分模型，其他模型还在适配中，具体支持列表和进度请前往：[github](https://github.com/PaddlePaddle/FastDeploy)

### RK3588 端

#### 编译与安装
说明：本文测试是基于 RK3588 Firefly Ubuntu20.04 v1.0.4a 固件，采用 python 方式安装与部署，python 版本为 3.8 。因为固件中自带 rknpu2 v1.4.0 环境，因此本文跳过了准备 rknpu2 部分。其他部署方式、准备 rknpu2 等内容请查看官方文档：
https://github.com/PaddlePaddle/FastDeploy/blob/develop/docs/cn/build_and_install/rknpu2.md

##### 准备环境
```bash
sudo apt update
sudo apt install -y python3 python3-dev python3-pip gcc python3-opencv python3-numpy
```
这里提供一个提前编译好的 python 包用于快速测试：[前往下载](https://pan.baidu.com/s/1z1KD-8McTj5mwmrozDD6pQ) 提取码：1234

使用这个包的话可以跳过编译步骤，直接进行安装（上一步准备环境还是需要的）。该包基于 FastDeploy v1.0.0 (commit id c4bb83ee)

<font color="red">注意：此项目处于快速迭代中，这里提供的预编译包可能已经过时，仅供快速了解、测试。实际开发使用请从最新源码编译：</font>
##### RK3588 上编译
```bash
sudo apt update
sudo apt install -y gcc cmake

git clone https://github.com/PaddlePaddle/FastDeploy.git

cd FastDeploy/python

export ENABLE_ORT_BACKEND=ON
export ENABLE_RKNPU2_BACKEND=ON
export ENABLE_VISION=ON
export RKNN2_TARGET_SOC=RK3588
python3 setup.py build
python3 setup.py bdist_wheel
```
如果你使用的 RK3588 内存比较小，编译可能会出现 OOM 错误，可以在`python3 setup.py build`命令后面追加`-j N`来控制编译线程数量。

##### RK3588 上安装
编译完成后可以在 FastDeploy/python/dist 下找到安装包，或者使用之前提供的预编译包。使用 pip3 安装
```bash
pip3 install fastdeploy_python-*-linux_aarch64.whl
```

#### 推理模型
在完成安装后，这里提供了3个已经调整好的 demo，[前往下载](https://pan.baidu.com/s/1z1KD-8McTj5mwmrozDD6pQ)，提取码：1234

<font color="red">注意：此项目处于快速迭代中，这里提供的模型可能已经过时，仅供快速了解、测试。实际开发使用请前往本文开头官方 github 获取最新内容</font>

解压后可以直接运行推理：
* Picodet 物体检测模型
```
cd demos/vision/detection/paddledetection/rknpu2/python

python3 infer.py --model_file ./picodet_s_416_coco_lcnet/picodet_s_416_coco_lcnet_rk3588.rknn \
                 --config_file ./picodet_s_416_coco_lcnet/infer_cfg.yml \
                 --image 000000014439.jpg
```

* Scrfd 人脸检测模型
```
cd demos/vision/facedet/scrfd/rknpu2/python

python3 infer.py --model_file ./scrfd_500m_bnkps_shape640x640_rk3588.rknn \
                 --image test_lite_face_detector_3.jpg
```

* PaddleSeg 人像分割模型
```
cd demos/vision/segmentation/paddleseg/rknpu2/python

python3 infer.py --model_file ./Portrait_PP_HumanSegV2_Lite_256x144_infer/Portrait_PP_HumanSegV2_Lite_256x144_infer_rk3588.rknn \
                 --config_file ./Portrait_PP_HumanSegV2_Lite_256x144_infer/deploy.yaml \
                 --image images/portrait_heng.jpg
```

### PC 端
上一章节在 RK3588 上进行部署的只是运行环境，demo 是提前在 PC 端转换与调整好的。也就是说进一步的开发例如模型转换、参数调整等，是需要在 PC 端进行，所以还需要在 x86_64 Linux PC 上安装 FastDeploy。

#### 编译与安装
需求：Ubuntu18 及以上，python 环境 3.6 或 3.8

##### 安装 rknn-toolkit2
建议使用 conda 或 virtualenv 新建一个虚拟环境进行安装！安装 conda 或 virtualenv 的方法百度有很多，这里跳过。

首先前往下载 rknn_toolkit2 安装包：[github](https://github.com/rockchip-linux/rknn-toolkit2/tree/master/packages)
```bash
sudo apt install -y libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 libgl1-mesa-glx libprotobuf-dev gcc g++

# 创建虚拟环境
conda create -n rknn2 python=3.6
conda activate rknn2

# rknn_toolkit2对numpy存在特定依赖,因此需要先安装numpy==1.16.6
pip3 install numpy==1.16.6

# 安装 rknn_toolkit2
pip3 install rknn_toolkit2-1.4.0-*cp36*-linux_x86_64.whl
```

##### 编译安装 FastDeploy
```bash
pip3 install wheel
sudo apt install -y cmake

git clone https://github.com/PaddlePaddle/FastDeploy.git
cd FastDeploy/python
export ENABLE_ORT_BACKEND=ON
export ENABLE_PADDLE_BACKEND=ON
export ENABLE_OPENVINO_BACKEND=ON
export ENABLE_VISION=ON
export ENABLE_TEXT=ON
# OPENCV_DIRECTORY可选，不指定会自动下载FastDeploy提供的预编译OpenCV库
export OPENCV_DIRECTORY=/usr/lib/x86_64-linux-gnu/cmake/opencv4

python setup.py build
python setup.py bdist_wheel

pip3 install dist/fastdeploy_python-*-linux_x86_64.whl
pip3 install paddlepaddle
```

#### 例子与教程
FastDeploy/examples 下有丰富的例子，每一层的 README.md 中都有详细的使用教程。