# NPU
There is a NPU module in RK3568, using this NPU module needs to download [RKNN SDK](http://en.t-firefly.com/doc/download/89.html#other_398) which provides programming interfaces for RK3566/RK3568 chip platforms with NPU. This SDK can help users deploy RKNN models exported by RKNN-Toolkit2 and accelerate the implementation of AI applications

## RKNN Model
RKNN is the model type used by the Rockchip NPU platform. It is a model file ending with the suffix `.rknn `. RKNN SDK provides a complete model transformation Python tool for users to convert their self-developed algorithm model into RKNN model

The RKNN model can run directly on the RK3568 platform. There are demos under `RKNN_API_for_RK356X_v1.1.0*/examples/`. Refer to the `README.md` to compile Android or Linux Demo (Need cross-compile environment). You can also just download compiled [demo](http://en.t-firefly.com/doc/download/89.html#other_401).

Run demo on the AIO-3568J as follows:
```shell
:/ # cd /data/rknn_ssd_demo_Android/    (Use rknn_ssd_demo_Linux in Linux)
:/data/rknn_ssd_demo_Android # chmod 777 rknn_ssd_demo
:/data/rknn_ssd_demo_Android # export LD_LIBRARY_PATH=./lib
:/data/rknn_ssd_demo_Android # ./rknn_ssd_demo model/ssd_inception_v2.rknn model/road.bmp
Loading model ...
rknn_init ...
model input num: 1, output num: 2
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

## Non-RKNN Model
For other models like Caffe, TensorFlow, etc, to run on RK3568 platform, conversions are needed. Use RKNN-Toolkit2 to convert other model into RKNN model.

## RKNN-Toolkit2
### Introduction of Tool
RKNN-Toolkit2 is a development kit that provides users with model conversion, inference and performance evaluation on PC and Rockchip NPU platforms. Users can easily complete the following functions through the Python interface provided by the tool:

* <font color=blue>**Model conversion**</font>: support to convert `Caffe` / `TensorFlow` / `TensorFlow Lite` / `ONNX` / `Darknet`
/ `PyTorch` model to RKNN model, support RKNN model import/export, which can be used on Rockchip NPU platform later
<br></br>
* <font color=blue>**Quantization**</font>: support to convert float model to quantization model, currently support quantized methods including asymmetric quantization(asymmetric_quantized-8, asymmetric_quantized-16). and support hybrid quantization. 
<font color=red>Asymmetric_quantized-16 not supported yet</font>
<br></br>
* <font color=blue>**Model inference**</font>: Able to simulate Rockchip NPU to run RKNN model on PC and get the inference result.
 This tool can also distribute the RKNN model to the specified NPU device to run, and get the inference results
<br></br>
* <font color=blue>**Performance evaluation**</font>: distribute the RKNN model to the specified NPU device to run, and evaluate the model performance in the actual device
<br></br>
* <font color=blue>**Memory evaluation**</font>: Evaluate memory consumption at runtime of the model. When using this function, 
the RKNN model must be distributed to the NPU device to run, and then call the relevant interface to obtain memory information
<br></br>
* <font color=blue>**Quantitative error analysis**</font>: This function will give the Euclidean or cosine distance of each layer of inference results before and after the model is quantized.
 This can be used to analyze how quantitative error occurs, and provide ideas for improving the accuracy of quantitative models

### Environment Dependence
* The system needs: Ubuntu 18.04 (x64) or later. <font color=red> The Toolkit can only be installed on PC, and Windows, MacOS or Debian not supported yet</font>
* Python version: 3.6
* Python rely on：
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

### RKNN-Toolkit2 installation

It is recommended to use virtualenv to manage the python environment because there may be multiple versions of the python environment in the system at the same time 
```shell
# 1）Install virtualenv、Python3.6 and pip3
sudo apt install virtualenv \
sudo apt-get install python3 python3-dev python3-pip
# 2）Install dependent libraries
sudo apt-get install libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
# 3）Use virtualenv and install Python dependency，such as requirements-1.1.0b0.txt
virtualenv -p /usr/bin/python3 venv
source venv/bin/activate
pip3 install -r doc/requirements*.txt
# 4）Install RKNN-Toolkit2，such as rknn_toolkit2-1.1.0b0-cp36-cp36m-linux_x86_64.whl
sudo pip3 install packages/rknn_toolkit2*.whl
# 5）Check if RKNN-Toolkit2 is installed successfully or not，press key ctrl+d to exit
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0$ python3
>>> from rknn.api import RKNN
>>>
```

The installation is successful if the import of RKNN module doesn't fail. One of the failures is as follows:
```shell
>>> from rknn.api import RKNN
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named 'rknn'
```

### Model Conversion Demo
Toolkit Demos are under `rknn-toolkit2-1.1.0*/examples`. Here we run a model conversion demo for example, this demo shows the process of converting tflite to RKNN, exporting model, inferencing, deploying on NPU and fetching results. For detailed implementation of the model conversion, please refer to the source code in Demo and the documents at the end of this page.

#### Simulate the running example on PC
RKNN-Toolkit2 has a built-in simulator, simply run a demo to deploy on NPU simulator.
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
Analysing :   0%|                                                            | 0Analysing :  84%|██████████████████████████████████████████▏       | 49/58 [00:0Analysing : 100%|██████████████████████████████████████████████████| 58/58 [00:00<00:00, 235.93it/s]
Quantizating :   0%|                                                         | 0Quantizating :   2%|▊                                                | 1/58 [00:Quantizating :  48%|███████████████████████▏                        | 28/58 [00:Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00, 117.84it/s]
I RKNN: librknnc version: 1.1.0b0 (8d7e25ad@2021-06-30T18:33:39)
I RKNN: set log level to 0
done
--> Export RKNN model
done
--> Init runtime environment
W init_runtime: target is None, use simulator!
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

#### Run on AIO-3568J NPU connected to the PC

RKNN Toolkit2 runs on the PC and connects to the AIO-3568J through the PC's USB. RKNN Toolkit2 transfers the RKNN model to the NPU device of AIO-3568J to run, and then obtains the inference results, performance information, etc. from the AIO-3568J

AIO-3568J Android need to refer to the section "[ADB USE](adb_use.html#preface)" open ADB，ADB is enabled by default on Linux. We could see adb device after opening
```shell
(venv) firefly@T-chip:~$ adb devices 
List of devices attached
XXXXXXXX	device
```

* First prepare AIO-3568J environment: update librknnrt.so and run rknn_server

*Android*
```shell
adb root && adb remount
adb push rknnrt/Android/rknn_server/arm64-v8a/vendor/bin/rknn_server /vendor/bin
adb push rknnrt/Android/librknn_api/arm64-v8a/librknnrt.so /vendor/lib64
adb push rknnrt/Android/librknn_api/arm64-v8a/librknnrt.so /vendor/lib

# run rknn_server on the serial terminal 
chmod +x /vendor/bin/rknn_server
setenforce 0
/vendor/bin/rknn_server 
```
*Linux*
```shell
adb push RKNN_SDK/Linux/rknn_server/aarch64/usr/bin/rknn_server /usr/bin/
adb push RKNN_SDK/Linux/librknn_api/aarch64/librknnrt.so /usr/lib/
adb push RKNN_SDK/Linux/librknn_api/aarch64/librknn_api.so /usr/lib/

# run rknn_server on the serial terminal
chmod +x /usr/bin/rknn_server
/usr/bin/rknn_server
```

* Then modify the demo file `examples/tflite/mobilenet_v1/test.py` on PC, add the target platform in it.
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

* Run test.py on host PC
```shell
(venv) firefly@T-chip:~/rknn-toolkit2-1.1.0b0/examples/tflite/mobilenet_v1$ python3 test.py 
--> config model
done
--> Loading model
done
--> Building model
Analysing : 100%|██████████████████████████████████████████████████| 58/58 [00:00<00:00, 186.60it/s]
Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00, 468.02it/s]
I RKNN: librknnc version: 1.1.0b0 (8d7e25ad@2021-06-30T18:33:39)
I RKNN: set log level to 0
done
--> Export RKNN model
done
--> Init runtime environment
I NPUTransfer: Starting NPU Transfer Client, Transfer version 2.1.0 (b5861e7@2020-11-23T11:50:36)
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

### Other Toolkit Demo
Other Toolkit demos can be found under `rknn-toolkit2-1.1.0*/examples/functions/`, such as quantization, accuracy analysis demos. For detailed implementation, please refer to the source code in Demo and the detailed development documents.

## Detailed Development Documents
Please refer to <<Rockchip_RK356X_User_Guide_RKNN_API_\*.pdf>> and <<Rockchip_User_Guide_RKNN_Toolkit2_\*.pdf>> in RKNN SDK for development.