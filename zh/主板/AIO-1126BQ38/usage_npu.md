# NPU使用

RV1126B 内置 NPU 模块, 处理性能最高可达3TOPS。使用该NPU需要下载[RKNN SDK](https://www.t-firefly.com/doc/download/367.html)，RKNN SDK 为带有 NPU 的 RK 系列芯片提供编程接口，能够帮助用户部署使用 RKNN-Toolkit2 导出的 RKNN 模型，加速 AI 应用的落地，具体支持平台列表可通过 SDK 中 README 文档或者访问[airockchip](https://github.com/airockchip/rknn-toolkit2/) 查看，开发建议以[airockchip](https://github.com/airockchip/rknn-toolkit2/)官方更新版本作为参考。

## RKNN 模型
RKNN 是 Rockchip NPU 平台使用的模型类型，以`.rknn`后缀结尾的模型文件。用户可以通过RKNN SDK提供的工具将自主研发的算法模型转换成 RKNN 模型

RKNN 模型可以直接运行在 RV1126B 平台上，在`rknpu2/examples`中有例子，根据`README.md`编译生成 Linux Demo（需要交叉编译环境）。也可以直接下载编译好的 [Demo](https://www.t-firefly.com/doc/download/367.html)。

在 AIO-1126BQ38 上运行 yolov5 demo 如下:
```shell
# Platform 为芯片平台，如RK3576，RK3588, RK3566_RK3568等
$ chmod 777 rknn_yolov5_demo
$ export LD_LIBRARY_PATH=./lib
$ ./rknn_yolov5_demo model/Platform/yolov5s-640-640.rknn ./model/bus.jpg resize ./out.jpg
post process config: box_conf_threshold = 0.25, nms_threshold = 0.45
Loading mode...
sdk version: 2.3.0 (c949ad889d@2024-11-07T11:35:33) driver version: 0.9.3
model input num: 1, output num: 3
  index=0, name=images, n_dims=4, dims=[1, 640, 640, 3], n_elems=1228800, size=1228800, w_stride = 640, size_with_stride=1228800, fmt=NHWC, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003922
  index=0, name=output0, n_dims=4, dims=[1, 255, 80, 80], n_elems=1632000, size=1632000, w_stride = 0, size_with_stride=1638400, fmt=NCHW, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003922
  index=1, name=286, n_dims=4, dims=[1, 255, 40, 40], n_elems=408000, size=408000, w_stride = 0, size_with_stride=409600, fmt=NCHW, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003922
  index=2, name=288, n_dims=4, dims=[1, 255, 20, 20], n_elems=102000, size=102000, w_stride = 0, size_with_stride=122880, fmt=NCHW, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003922
model is NHWC input fmt
model input height=640, width=640, channel=3
Read ./model/bus.jpg ...
img width = 640, img height = 640
once run use 67.055000 ms
loadLabelName ./model/coco_80_labels_list.txt
person @ (209 243 286 510) 0.879723
person @ (479 238 560 526) 0.870588
person @ (109 238 231 534) 0.839831
bus @ (91 129 555 464) 0.692042
person @ (79 353 121 517) 0.300961
save detect result to ./out.jpg
loop count = 10 , average run  59.530100 ms
```

## 非 RKNN 模型
对于 Caffe、TensorFlow 等其他模型，想要在 RV1126B 平台运行，需要先进行模型转换。可以使用 RKNN-Toolkit2 工具将模型转换成 RKNN 格式。
## RKNN-Toolkit2工具
### 工具介绍
RKNN-Toolkit2 是为用户提供在 PC、Rockchip NPU 平台上进行模型转换、推理和性能评估的开发套件，用户通过该工具提供的 Python 接口可以便捷地完成各种操作。

工具的全部功能简单介绍如下：

* <font color=blue>**模型转换**</font>: 支持 `Caffe`、`TensorFlow`、`TensorFlow Lite`、`ONNX`、`DarkNet`、`PyTorch` 等模型转为 RKNN 模型，并支持 RKNN 模型导入导出，RKNN 模型能够在 Rockchip NPU 平台上加载使用
<br></br>
*  <font color=blue>**量化功能**</font>: 支持将浮点模型量化为定点模型，目前支持的量化方法为非对称量化
( asymmetric_quantized-8 及 asymmetric_quantized-16 )，并支持混合量化功能 。
<font color=red>asymmetric_quantized-16 目前版本暂不支持</font>
<br></br>
* <font color=blue>**模型推理**</font>: 能够在 PC 上模拟 Rockchip NPU 运行 RKNN 模型并获取推理结果; 或将 RKNN
模型分发到指定的 NPU 设备上进行推理并获取推理结果
<br></br>
* <font color=blue>**性能评估**</font>: 将 RKNN 模型分发到指定 NPU 设备上运行，以评估模型在实际设备上运行时的性能
<br></br>
* <font color=blue>**内存评估**</font>: 评估模型运行时的内存的占用情况。使用该功能时，必须将 RKNN 模型分发到 NPU 设备中运行，并调用相关接口获取内存使用信息
<br></br>
*  <font color=blue>**量化精度分析**</font>: 该功能将给出模型量化前后每一层推理结果与浮点模型推理结果的余弦距离，以便于分析量化误差是如何出现的，为提高量化模型的精度提供思路

### 环境依赖
* 系统依赖：RKNN-Toolkit2 目前版本适用系统`Ubuntu18.04(x64)`及以上，<font color=red>工具只能安装在 PC 上，暂不支持 Windows、MacOS、Debian 等操作系统</font>
* Python版本：3.6/3.8
* Python依赖库：
```shell
#Python3.6
cat doc/requirements_cp36-1.3.0.txt
numpy==1.16.6
onnx==1.7.0
onnxoptimizer==0.1.0
onnxruntime==1.6.0
tensorflow==1.14.0
tensorboard==1.14.0
protobuf==3.12.0
torch==1.6.0
torchvision==0.7.0
psutil==5.6.2
ruamel.yaml==0.15.81
scipy==1.2.1
tqdm==4.27.0
requests==2.21.0
opencv-python==4.4.0.46
PuLP==2.4
scikit_image==0.17.2
# if install bfloat16 failed, please install numpy manually first. "pip install numpy==1.16.6"
bfloat16==1.1
flatbuffers==1.12

#Python3.8
cat doc/requirements_cp38-1.3.0.txt
numpy==1.17.3
onnx==1.7.0
onnxoptimizer==0.1.0
onnxruntime==1.6.0
tensorflow==2.2.0
tensorboard==2.2.2
protobuf==3.12.0
torch==1.6.0
torchvision==0.7.0
psutil==5.6.2
ruamel.yaml==0.15.81
scipy==1.4.1
tqdm==4.27.0
requests==2.21.0
opencv-python==4.4.0.46
PuLP==2.4
scikit_image==0.17.2
# if install bfloat16 failed, please install numpy manually first. "pip install numpy==1.17.3"
bfloat16==1.1
```

### RKNN-Toolkit2 安装

建议使用 virtualenv 管理 Python 环境，因为系统中可能同时有多个版本的 Python 环境，以 Python3.6 为例
```shell
# 1）安装virtualenv 环境、Python3.6 和 pip3
sudo apt-get install virtualenv \
sudo apt-get install python3 python3-dev python3-pip
# 2）安装相关依赖
sudo apt-get install libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
# 3）使用 virtualenv 管理 Python 环境并安装 Python 依赖，Python3.6用requirements_cp36-1.3.0.txt
virtualenv -p /usr/bin/python3 venv
source venv/bin/activate
pip3 install -r doc/requirements_cp36-*.txt
# 4）安装 RKNN-Toolkit2，如rknn_toolkit2-1.3.0_11912b58-cp36-cp36m-linux_x86_64.whl
sudo pip3 install packages/rknn_toolkit2*cp36*.whl
# 5）检查RKNN-Toolkit2是否安装成功，可按ctrl+d组合键退出
(venv) firefly@T-chip:~/rknn-toolkit2$ python3
>>> from rknn.api import RKNN
>>>
```

如果导入 RKNN 模块没有失败，说明安装成功，失败情况之一如下：
```shell
>>> from rknn.api import RKNN
Traceback (most recent call last):
  File "<stdin>"，line 1，in <module>
ImportError: No module named 'rknn'
```

### 模型转换 Demo
在`rknn-toolkit2/examples`下有各种功能的 Toolkit Demo ，这里我们运行一个模型转换 Demo 为例子，这个 Demo 展示了在 PC 上将 tflite 模型转换成 RKNN 模型，然后导出、推理、部署到 NPU 平台运行并取回结果的过程。模型转换的具体实现请参考 Demo 内源代码以及本页末尾的文档。

#### 在 PC 上仿真运行

* RKNN-Toolkit2 自带了一个模拟器，直接在 PC 上运行 Demo 即是将转换后的模型部署到仿真 NPU 上运行
```shell
(venv) firefly@T-chip:~/rknn-toolkit2$ cd examples/tflite/mobilenet_v1
(venv) firefly@T-chip:~/rknn-toolkit2/examples/tflite/mobilenet_v1$ ls
dataset.txt  dog_224x224.jpg  mobilenet_v1_1.0_224.tflite  test.py
(venv) firefly@T-chip:~/rknn-toolkit2/examples/tflite/mobilenet_v1$ python3 test.py
W __init__: rknn-toolkit2 version: 1.3.0-11912b58
--> Config model
W config: 'target_platform' is None, use rk3566 as default, Please set according to the actual platform!
done
--> Loading model
INFO: Initialized TensorFlow Lite runtime.
done
--> Building model
Analysing : 100%|█████████████████████████████████████████████████| 58/58 [00:00<00:00, 1869.33it/s]
Quantizating : 100%|████████████████████████████████████████████████| 58/58 [00:00<00:00, 68.07it/s]
W build: The default input dtype of 'input' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!
done
--> Export rknn model
done
--> Init runtime environment
Analysing : 100%|█████████████████████████████████████████████████| 60/60 [00:00<00:00, 1434.93it/s]
Preparing : 100%|██████████████████████████████████████████████████| 60/60 [00:00<00:00, 373.17it/s]
W init_runtime: target is None, use simulator!
done
--> Running model
mobilenet_v1
-----TOP 5-----
[156]: 0.9345703125
[155]: 0.0570068359375
[205]: 0.00429534912109375
[284]: 0.003116607666015625
[285]: 0.00017178058624267578

done
```

#### 运行在与 PC 相连的 AIO-1126BQ38 NPU 平台上

RKNN-Toolkit2 通过 PC 的 USB 连接到 OTG 设备 AIO-1126BQ38。RKNN-Toolkit2
将 RKNN 模型传到 AIO-1126BQ38 的 NPU 上运行，再从 AIO-1126BQ38 上获取推理结果、性能信息等：


* 首先部署 AIO-1126BQ38 环境：更新librknnrt.so及运行rknn_server 

```shell
adb push rknpu2/runtime/Linux/rknn_server/aarch64/usr/bin/rknn_server /usr/bin/
adb push rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so /usr/lib/
adb push rknpu2/runtime/Linux/librknn_api/aarch64/librknn_api.so /usr/lib/

# 可以使用 "systemctl status rknn_server" 查看rknn_server服务是否处于运行状态
# 若没有运行，请在板子的串口终端运行rknn_server
chmod +x /usr/bin/rknn_server
/usr/bin/rknn_server
```

* 然后在 PC 上修改`examples/tflite/mobilenet_v1/test.py`文件，在其中添加目标平台
```shell
diff --git a/examples/tflite/mobilenet_v1/test.py b/examples/tflite/mobilenet_v1/test.py
index 0507edb..fd2e070 100755
--- a/examples/tflite/mobilenet_v1/test.py
+++ b/examples/tflite/mobilenet_v1/test.py
@@ -24,11 +24,11 @@ def show_outputs(outputs):
 if __name__ == '__main__':
 
     # Create RKNN object
-    rknn = RKNN(verbose=True)
+    rknn = RKNN()
 
     # Pre-process config
     print('--> Config model')
-    rknn.config(mean_values=[128, 128, 128], std_values=[128, 128, 128])
+    rknn.config(mean_values=[128, 128, 128], std_values=[128, 128, 128], target_platform='rv1126b')
     print('done')
 
     # Load model
@@ -62,7 +62,7 @@ if __name__ == '__main__':
 
     # Init runtime environment
     print('--> Init runtime environment')
-    ret = rknn.init_runtime()
+    ret = rknn.init_runtime(target='rv1126b')
     if ret != 0:
         print('Init runtime environment failed!')
         exit(ret)
```

* PC 端运行test.py
```shell
(venv) firefly@T-chip:~/rknn-toolkit2/examples/tflite/mobilenet_v1$ python3 test.py 
W __init__: rknn-toolkit2 version: 1.3.0-11912b58
--> Config model
done
--> Loading model
INFO: Initialized TensorFlow Lite runtime.
done
--> Building model
Analysing : 100%|█████████████████████████████████████████████████| 58/58 [00:00<00:00, 1903.99it/s]
Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00, 114.55it/s]
W build: The default input dtype of 'input' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!
done
--> Export rknn model
done
--> Init runtime environment
I NPUTransfer: Starting NPU Transfer Client, Transfer version 2.1.0 (b5861e7@2020-11-23T11:50:36)
D RKNNAPI: ==============================================
D RKNNAPI: RKNN VERSION:
D RKNNAPI:   API: 1.3.0 (121b661 build: 2022-04-29 11:07:20)
D RKNNAPI:   DRV: rknn_server: 1.3.0 (121b661 build: 2022-04-29 11:11:57)
D RKNNAPI:   DRV: rknnrt: 1.3.0 (c193be371@2022-05-04T20:16:22)
D RKNNAPI: ==============================================
done
--> Running model
mobilenet_v1
-----TOP 5-----
[156]: 0.93505859375
[155]: 0.057037353515625
[205]: 0.0038814544677734375
[284]: 0.0031185150146484375
[285]: 0.00017189979553222656

done
```
### 其他 Toolkit Demo
其他 Toolkit Demo 可以在`rknn-toolkit2/examples/`下找到，例如量化、精度评估等。具体实现以及使用方法请参考 Demo 内源代码以及详细开发文档。

## 详细开发文档
AIO-1126BQ38 NPU 及 Toolkit 详细使用方法请参考RKNN SDK下《Rockchip_RKNPU_User_Guide_RKNN_API_\*.pdf》、《Rockchip_User_Guide_RKNN_Toolkit2_\*.pdf》文档。

