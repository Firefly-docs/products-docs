# NPU
RV1126B has a NPU(*Neural Process Unit*) that Neural network acceleration engine with processing performance up to 3 TOPS. Using this NPU module needs to download [RKNN SDK](https://en.t-firefly.com/doc/download/346.html) which provides programming interfaces for RK series chips platforms with NPU. This SDK can help users deploy RKNN models exported by RKNN-Toolkit2 and accelerate the implementation of AI applications. The specific list of supported platforms can be found in the README document included with the SDK or by visiting [airockchip](https://github.com/airockchip/rknn-toolkit2/). It is recommended to refer to the official updates provided by [airockchip](https://github.com/airockchip/rknn-toolkit2/) for development purposes.

## RKNN Model
RKNN is the model type used by the Rockchip NPU platform. It is a model file ending with the suffix `.rknn `. RKNN SDK provides a complete model transformation Python tool for users to convert their self-developed algorithm model into RKNN model

The RKNN model can run directly on the RV1126B platform. There are demos under `rknpu2/examples`. Refer to the `README.md` to compile Android or Linux Demo (Need cross-compile environment). You can also just download compiled [demo](https://en.t-firefly.com/doc/download/346.html).

Run yolov5 demo on the AIO-1126BJD4 as follows:
```shell
# Platform is a chip platform, such as RK3576, RK3588, RK3566-RK3568, etc

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

## Non-RKNN Model
For other models like Caffe, TensorFlow, etc, to run on RV1126B platform, conversions are needed. Use RKNN-Toolkit2 to convert other model into RKNN model.

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
* Python version: 3.6/3.8
* Python rely on：
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

### RKNN-Toolkit2 installation

It is recommended to use virtualenv to manage the python environment because there may be multiple versions of the python environment in the system at the same time. Let's start with Python 3.6 as an example: 
```shell
# 1）Install virtualenv、Python3.6 and pip3
sudo apt install virtualenv \
sudo apt-get install python3 python3-dev python3-pip
# 2）Install dependent libraries
sudo apt-get install libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
# 3）Use virtualenv and install Python dependency, such as requirements_cp36-1.3.0.txt 
virtualenv -p /usr/bin/python3 venv
source venv/bin/activate
pip3 install -r doc/requirements_cp36-*.txt
# 4）Install RKNN-Toolkit2，such as rknn_toolkit2-1.3.0_11912b58-cp36-cp36m-linux_x86_64.whl
sudo pip3 install packages/rknn_toolkit2*cp36*.whl
# 5）Check if RKNN-Toolkit2 is installed successfully or not，press key ctrl+d to exit
(venv) firefly@T-chip:~/rknn-toolkit2$ python3
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
Toolkit Demos are under `rknn-toolkit2/examples`. Here we run a model conversion demo for example, this demo shows the process of converting tflite to RKNN, exporting model, inferencing, deploying on NPU and fetching results. For detailed implementation of the model conversion, please refer to the source code in Demo and the documents at the end of this page.

#### Simulate the running example on PC
RKNN-Toolkit2 has a built-in simulator, simply run a demo to deploy on NPU simulator.
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

#### Run on AIO-1126BJD4 NPU connected to the PC

RKNN Toolkit2 runs on the PC and connects to the AIO-1126BJD4 through the PC's USB. RKNN Toolkit2 transfers the RKNN model to the NPU device of AIO-1126BJD4 to run, and then obtains the inference results, performance information, etc. from the AIO-1126BJD4


* First prepare AIO-1126BJD4 environment: update librknnrt.so and run rknn_server


*Linux*
```shell
adb push rknpu2/runtime/Linux/rknn_server/aarch64/usr/bin/rknn_server /usr/bin/
adb push rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so /usr/lib/

# We can use "systemctl status rknn_server" to check whether the rknn_server is running.
# Without running rknn_server, run it on the serial terminal
chmod +x /usr/bin/rknn_server
/usr/bin/rknn_server
```

* Then modify the demo file `examples/tflite/mobilenet_v1/test.py` on PC, add the target platform in it.
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

* Run test.py on host PC
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

### Other Toolkit Demo
Other Toolkit demos can be found under `rknn-toolkit2/examples/`, such as quantization, accuracy analysis demos. For detailed implementation, please refer to the source code in Demo and the detailed development documents.

## Detailed Development Documents
Please refer to <<Rockchip_RKNPU_User_Guide_RKNN_API_\*.pdf>> and <<Rockchip_User_Guide_RKNN_Toolkit2_\*.pdf>> in RKNN SDK for development.

