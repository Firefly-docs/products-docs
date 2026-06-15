# NPU使用

RK3568 内置 NPU 模块。使用该NPU需要下载[RKNN SDK](https://www.t-firefly.com/doc/download/103.html#other_477)，RKNN SDK 为带有 NPU 的 RK3566/RK3568 芯片平台提供编程接口，能够帮助用户部署使用 RKNN-Toolkit2 导出的 RKNN 模型，加速 AI 应用的落地

## RKNN 模型
RKNN 是 Rockchip NPU 平台使用的模型类型，以`.rknn`后缀结尾的模型文件。用户可以通过RKNN SDK提供的工具将自主研发的算法模型转换成 RKNN 模型

RKNN 模型可以直接运行在 RK3568 平台上，在`RKNN_API_for_RK356X_v1.1.0*/examples/`中有例子，根据`README.md`编译生成 Android 或 Linux Demo（需要交叉编译环境）。也可以直接下载编译好的 [Demo](https://www.t-firefly.com/doc/download/103.html#other_478)。

在 ROC-RK3568-PC-SE 上运行demo如下:
```shell
:/ # cd /data/rknn_ssd_demo_Android/    (Linux 系统使用 rknn_ssd_demo_Linux 即可)
:/data/rknn_ssd_demo_Android # chmod 777 rknn_ssd_demo
:/data/rknn_ssd_demo_Android # export LD_LIBRARY_PATH=./lib
:/data/rknn_ssd_demo_Android # ./rknn_ssd_demo model/ssd_inception_v2.rknn model/road.bmp
Loading model ...
rknn_init ...
model input num: 1，output num: 2
input tensors:
index=0 name=Preprocessor/sub:0 n_dims=4 dims=[3 300 300 1] n_elems=270000 size=270000 fmt=0 type=2 qnt_type=2 fl=0 zp=0 scale=0.007812
output tensors:
index=0 name=concat:0 n_dims=4 dims=[4 1 1917 1] n_elems=7668 size=7668 fmt=0 type=2 qnt_type=2 fl=0 zp=53 scale=0.089455
index=1 name=concat_1:0 n_dims=4 dims=[1 91 1917 1] n_elems=174447 size=174447 fmt=0 type=2 qnt_type=2 fl=0 zp=53 scale=0.143593
rknn_run
loadLabelName
ssd - loadLabelName ./model/coco_labels_list.txt
loadBoxPriors
person @ (13 125 59 212) 0.982374
person @ (110 119 152 197) 0.969119
bicycle @ (171 165 278 234) 0.969119
person @ (206 113 256 216) 0.964519
car @ (146 133 216 170) 0.959264
person @ (49 133 58 156) 0.606060
person @ (83 134 92 158) 0.606060
person @ (96 135 106 162) 0.464163
```

## 非 RKNN 模型
对于 Caffe、TensorFlow 等其他模型，想要在 RK3568 平台运行，需要先进行模型转换。可以使用 RKNN-Toolkit2 工具将模型转换成 RKNN 格式。
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
* Python版本：3.6
* Python依赖库：
```shell
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
tflite==2.3.0
opencv-python==4.4.0.46
PuLP==2.4
scikit_image==0.17.2
```

### RKNN-Toolkit2 安装

建议使用 virtualenv 管理 Python 环境，因为系统中可能同时有多个版本的 Python 环境
```shell
# 1）安装virtualenv 环境、Python3.6 和 pip3
sudo apt install virtualenv \
sudo apt-get install python3 python3-dev python3-pip
# 2）安装相关依赖
sudo apt-get install libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
# 3）使用 virtualenv 管理 Python 环境并安装 Python 依赖，如requirements-1.1.0b0.txt
virtualenv -p /usr/bin/python3 venv
source venv/bin/activate
pip3 install -r doc/requirements*.txt
# 4）安装 RKNN-Toolkit2，如rknn_toolkit2-1.1.0b0-cp36-cp36m-linux_x86_64.whl
sudo pip3 install packages/rknn_toolkit2*.whl
# 5）检查RKNN-Toolkit2是否安装成功，可按ctrl+d组合键退出
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0$ python3
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
在`rknn-toolkit2-1.1.0*/examples`下有各种功能的 Toolkit Demo ，这里我们运行一个模型转换 Demo 为例子，这个 Demo 展示了在 PC 上将 tflite 模型转换成 RKNN 模型，然后导出、推理、部署到 NPU 平台运行并取回结果的过程。模型转换的具体实现请参考 Demo 内源代码以及本页末尾的文档。

#### 在 PC 上仿真运行

* RKNN-Toolkit2 自带了一个模拟器，直接在 PC 上运行 Demo 即是将转换后的模型部署到仿真 NPU 上运行
```shell
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0$ cd examples/tflite/mobilenet_v1
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0/examples/tflite/mobilenet_v1$ ls
dataset.txt  dog_224x224.jpg  mobilenet_v1_1.0_224.tflite  test.py
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0/examples/tflite/mobilenet_v1$ python3 test.py 
--> config model
done
--> Loading model
done
--> Building model
Analysing :   0%|                                                            | 0Analysing :  84%|██████████████████████████████████████████▏       | 49/58 [00:0Analysing : 100%|██████████████████████████████████████████████████| 58/58 [00:00<00:00，235.93it/s]
Quantizating :   0%|                                                         | 0Quantizating :   2%|▊                                                | 1/58 [00:Quantizating :  48%|███████████████████████▏                        | 28/58 [00:Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00，117.84it/s]
I RKNN: librknnc version: 1.1.0b0 (8d7e25ad@2021-06-30T18:33:39)
I RKNN: set log level to 0
done
--> Export RKNN model
done
--> Init runtime environment
W init_runtime: target is None，use simulator!
done
--> Running model
I RKNN: librknnc version: 1.1.0b0 (8d7e25ad@2021-06-30T18:33:39)
I RKNN: set log level to 0
mobilenet_v1
-----TOP 5-----
[156]: 0.84228515625
[155]: 0.08807373046875
[205]: 0.01416015625
[284]: 0.0082550048828125
[194 260]: 0.0028209686279296875
```

#### 运行在与 PC 相连的 ROC-RK3568-PC-SE NPU 平台上

RKNN-Toolkit2 通过 PC 的 USB 连接到 OTG 设备 ROC-RK3568-PC-SE。RKNN-Toolkit2
将 RKNN 模型传到 ROC-RK3568-PC-SE 的 NPU 上运行，再从 ROC-RK3568-PC-SE 上获取推理结果、性能信息等：


* 首先部署 ROC-RK3568-PC-SE 环境：更新librknnrt.so及运行rknn_server

*Android*
```shell
adb root && adb remount
adb push RKNN_SDK/Android/rknn_server/arm64-v8a/vendor/bin/rknn_server /vendor/bin
adb push RKNN_SDK/Android/librknn_api/arm64-v8a/librknnrt.so /vendor/lib64
adb push RKNN_SDK/Android/librknn_api/arm64-v8a/librknnrt.so /vendor/lib

# 在板子的串口终端运行rknn_server
chmod +x /vendor/bin/rknn_server
setenforce 0
/vendor/bin/rknn_server 
```
*Linux*
```shell
adb push RKNN_SDK/Linux/rknn_server/aarch64/usr/bin/rknn_server /usr/bin/
adb push RKNN_SDK/Linux/librknn_api/aarch64/librknnrt.so /usr/lib/
adb push RKNN_SDK/Linux/librknn_api/aarch64/librknn_api.so /usr/lib/

# 在板子的串口终端运行rknn_server
chmod +x /usr/bin/rknn_server
/usr/bin/rknn_server
```

* 然后在 PC 上修改`examples/tflite/mobilenet_v1/test.py`文件，在其中添加目标平台
```shell
diff --git a/test.py b/test.py
index 61ad668..51a01e2 100644
--- a/test.py
+++ b/test.py
@@ -62,7 +62,7 @@ if __name__ == '__main__':
 
     # init runtime environment
     print('--> Init runtime environment')
-    ret = rknn.init_runtime()
+    ret = rknn.init_runtime(target='rk3568')
     if ret != 0:
         print('Init runtime environment failed')
         exit(ret)
```

* PC 端运行test.py
```shell
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0/examples/tflite/mobilenet_v1$ python3 test.py 
--> config model
done
--> Loading model
done
--> Building model
Analysing : 100%|██████████████████████████████████████████████████| 58/58 [00:00<00:00，186.60it/s]
Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00，468.02it/s]
I RKNN: librknnc version: 1.1.0b0 (8d7e25ad@2021-06-30T18:33:39)
I RKNN: set log level to 0
done
--> Export RKNN model
done
--> Init runtime environment
I NPUTransfer: Starting NPU Transfer Client，Transfer version 2.1.0 (b5861e7@2020-11-23T11:50:36)
D RKNNAPI: ==============================================
D RKNNAPI: RKNN VERSION:
D RKNNAPI:   API: 1.1.0b0 (ccc3bbc build: 2021-06-30 20:30:36)
D RKNNAPI:   DRV: 1.1.0b0 (74e78f5 build: 2021-06-30 20:09:41)
D RKNNAPI: ==============================================
done
--> Running model
mobilenet_v1
-----TOP 5-----
[156]: 0.84228515625
[155]: 0.08807373046875
[205]: 0.01415252685546875
[284]: 0.0082550048828125
[194 260]: 0.0028209686279296875

done
```
### 其他 Toolkit Demo
其他 Toolkit Demo 可以在`rknn-toolkit2-1.1.0*/examples/functions/`下找到，例如量化、精度评估等。具体实现以及使用方法请参考 Demo 内源代码以及详细开发文档。

## 详细开发文档
ROC-RK3568-PC-SE NPU 及 Toolkit 详细使用方法请参考RKNN SDK下《Rockchip_RK356X_User_Guide_RKNN_API_\*.pdf》、《Rockchip_User_Guide_RKNN_Toolkit2_\*.pdf》文档。