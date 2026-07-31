# NPU usage

RV1106 has a built-in NPU module, with a processing performance of up to 0.5TOPS. Using this NPU requires downloading tools.

NPU model conversion tool download address:

```
https://github.com/rockchip-linux/rknn-toolkit2
```

NPU runtime library download address:

```
https://github.com/airockchip/rknn-toolkit2/tree/master/rknpu2
```

These tools can help users deploy RKNN models exported using RKNN-Toolkit2 and accelerate the implementation of AI applications.

## RKNN model
RKNN is the model type used by the Rockchip NPU platform, and the model file ends with the `.rknn` suffix. Users can convert self-developed algorithm models into RKNN models through the tools provided by rknn-toolkit2.

The RKNN model can be run directly on the RV1106 platform. There are examples in `rknn-toolkit2/rknpu2/examples/RV1106_RV1103`. Compile and generate Linux Demo according to `README.md` (requires cross-compilation environment). 

* CT36L Operating Instructions

Since the default SPI FALSH memory of CT36L is relatively small, it is not enough to store the rknn model to test the npu. **You need to insert the SD card into the SD card slot on the device to store the rknn model**.

* Insert the SD card into the SD card slot on the board and execute the following command to mount the SD card:

```bash
//Create a mnt mounting directory in the /userdata/ directory
# mkdir -p /userdata/mnt

// If the SD card is automatically mounted, please unmount it first
# umount /mnt/sdcard/

//Mount the SD card device to the mnt directory. Note that the -o exec parameter must be added to mount the SD card here. Without adding this parameter, you will not be able to give running permission to rknn demo.
# mount -o exec /dev/mmcblk1 /userdata/mnt
```

* Cross-compile on PC to generate rknn demo

```shell
// Declare the file directory where the tool chain is located /path/to/sdk Replace it with the corresponding project directory
export RK_RV1106_TOOLCHAIN=/path/to/sdk/T36/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/bin/arm-rockchip830-linux-uclibcgnueabih

// Enter the rknn demo directory
cd rknn-toolkit2/rknpu2/examples/RV1106_RV1103/rknn_yolov5_demo

// Cross compile rknn demo
./build-linux_RV1106.sh

// Connect the board with a usb cable and push the compiled rknn demo to
adb push install/rknn_yolov5_demo_Linux/ /userdata/mnt/
```

* Run the demo on CT36L/CT36B as follows:

```bash
//Declare the linked library file directory
# export LD_LIBRARY_PATH=/userdata/mnt/rknn_yolov5_demo_Linux/lib

// Enter the rknn_yolov5_demo_Linux directory
# cd /userdata/mnt/rknn_yolov5_demo_Linux

//Give execution permissions
# chmod 777 rknn_yolov5_demo

//Run demo to test npu
# ./rknn_yolov5_demo model/RV1106/yolov5s-640-640.rknn ./model/bus.jpg 
rknn_api/rknnrt version: 1.5.2 (c6b7b351a@2023-08-23T15:29:48), driver version: 0.7.2
model input num: 1, output num: 3
input tensors:
  index=0, name=images, n_dims=4, dims=[1, 640, 640, 3], n_elems=1228800, size=1228800, fmt=NHWC, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003922
output tensors:
  index=0, name=output, n_dims=4, dims=[1, 80, 80, 255], n_elems=1632000, size=1632000, fmt=NHWC, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003860
  index=1, name=283, n_dims=4, dims=[1, 40, 40, 255], n_elems=408000, size=408000, fmt=NHWC, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003922
  index=2, name=285, n_dims=4, dims=[1, 20, 20, 255], n_elems=102000, size=102000, fmt=NHWC, type=INT8, qnt_type=AFFINE, zp=-128, scale=0.003915
custom string: 
Begin perf ...
   0: Elapse Time = 120.09ms, FPS = 8.33
model is NHWC input fmt
loadLabelName ./model/coco_80_labels_list.txt
person @ (208 244 286 506) 0.884136
person @ (479 238 560 526) 0.863766
person @ (110 236 230 535) 0.832498
bus @ (94 130 553 464) 0.697389
person @ (79 354 122 516) 0.349307
```
* CT36B Operating Instructions

Since the default EMMC memory of CT36B is relatively large, it is enough to store the rknn model to test the npu. Directly refer to the above-mentioned "Cross-compiling to generate rknn demo on PC" chapter to operate.

## Non-RKNN model

For other models such as Caffe and TensorFlow, if you want to run them on the RV1106 platform, you need to convert the model first. The model can be converted to RKNN format using the RKNN-Toolkit2 tool.
## RKNN-Toolkit2 tool
### Tool introduction
RKNN-Toolkit2 is a development kit that provides users with model conversion, inference and performance evaluation on PC and Rockchip NPU platforms. Users can easily complete various operations through the Python interface provided by the tool.

All functions of the tool are briefly introduced as follows:

* <font color=blue>**Model conversion**</font>: Supports the conversion of `Caffe`, `TensorFlow`, `TensorFlow Lite`, `ONNX`, `DarkNet`, `PyTorch` and other models into RKNN models, It also supports RKNN model import and export. The RKNN model can be loaded and used on the Rockchip NPU platform.
<br></br>
* <font color=blue>**Quantization function**</font>: Supports quantizing floating-point models into fixed-point models. The currently supported quantization method is asymmetric quantization.
(asymmetric_quantized-8 and asymmetric_quantized-16), and supports hybrid quantization functions.
<font color=red>asymmetric_quantized-16 is not supported in the current version</font>
<br></br>
* <font color=blue>**Model Inference**</font>: Ability to simulate Rockchip NPU on PC to run RKNN model and obtain inference results; or convert RKNN to
Distribute the model to the designated NPU device for inference and obtain the inference results
<br></br>
* <font color=blue>**Performance Evaluation**</font>: Distribute the RKNN model to the specified NPU device to evaluate the performance of the model when running on the actual device.
<br></br>
* <font color=blue>**Memory Evaluation**</font>: Evaluate the memory usage when the model is running. When using this function, the RKNN model must be distributed to the NPU device for running, and the relevant interfaces must be called to obtain memory usage information.
<br></br>
* <font color=blue>**Quantization Accuracy Analysis**</font>: This function will give the cosine distance between the inference results of each layer before and after model quantization and the floating-point model inference results, so as to facilitate the analysis of how quantization errors occur. , providing ideas for improving the accuracy of quantitative models

### Environment dependencies
* System dependency: The current version of RKNN-Toolkit2 is suitable for system `Ubuntu18.04(x64)` and above. <font color=red>The tool can only be installed on PC and does not currently support Windows, MacOS, Debian and other operating systems</font >
* Python version: 3.6/3.8
* Python dependent libraries:
```shell
#Python3.8
cat rknn-toolkit2/packages/requirements_cp38-1.6.0.txt
# if install failed, please change the pip source to 'https://mirror.baidu.com/pypi/simple'

# base deps
protobuf==3.20.3

# utils
psutil>=5.9.0
ruamel.yaml>=0.17.4
scipy>=1.5.4
tqdm>=4.64.0
opencv-python>=4.5.5.64
fast-histogram>=0.11

# base
onnx==1.14.1
onnxoptimizer==0.2.7
onnxruntime==1.16.0
torch==1.10.1
tensorflow==2.8.0
```
### RKNN-Toolkit2 installation

It is recommended to use virtualenv to manage the Python environment, because there may be multiple versions of Python environments in the system at the same time, taking Python3.8 as an example
```shell
# 1) Install virtualenv environment, Python3.8 and pip3
sudo apt-get install virtualenv \
sudo apt-get install python3 python3-dev python3-pip
# 2) Install related dependencies
sudo apt-get install libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
# 3) Use virtualenv to manage the Python environment and install Python dependencies. Python3.8 uses requirements_cp38-1.3.0.txt
virtualenv -p /usr/bin/python3 venv
source venv/bin/activate
pip3 install -r ./rknn-toolkit2/packages/requirements_cp38-1.6.0.txt
# 4) Check whether RKNN-Toolkit2 is installed successfully. You can press ctrl+d key combination to exit.
(venv) firefly@T-chip:~/rknn-toolkit2$ python3
>>> from rknn.api import RKNN
>>>
```

If there is no failure to import the RKNN module, the installation is successful. One of the failure conditions is as follows:
```shell
>>> from rknn.api import RKNN
Traceback (most recent call last):
   File "<stdin>", line 1, in <module>
ImportError: No module named 'rknn'
```

### Model conversion Demo
There are Toolkit Demo with various functions under `rknn-toolkit2/examples`. Here we run a model conversion Demo as an example. This Demo shows the conversion of the tflite model into the RKNN model on the PC, and then exports, infers, and deploys it to the NPU. The process by which the platform runs and gets back results. For the specific implementation of model conversion, please refer to the source code in the Demo and the documentation at the end of this page.

#### Simulate and run on PC

* RKNN-Toolkit2 comes with a simulator. Running the Demo directly on the PC means deploying the converted model to the simulated NPU for running.

```shell
(.venv) lvsx@lvsx:~/rv1106/rknn-toolkit2/examples/tflite/mobilenet_v1$ python3 test.py
W __init__: rknn-toolkit2 version: 1.5.2+b642f30c
--> Config model
done
--> Loading model
2023-12-22 15:13:10.185671: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcudart.so.11.0'; dlerror: libcudart.so.11.0: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /home/lvsx/.venv/lib/python3.8/site-packages/cv2/../../lib64:
2023-12-22 15:13:10.185694: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
Loading : 100%|██████████████████████████████████████████████████| 58/58 [00:00<00:00, 16371.87it/s]
done
--> Building model
I base_optimize ...
I base_optimize done.
I 
I fold_constant ...
I fold_constant done.
I 
I correct_ops ...
I correct_ops done.
I 
I fuse_ops ...
I fuse_ops results:
I     convert_squeeze_to_reshape: remove node = ['MobilenetV1/Logits/SpatialSqueeze'], add node = ['MobilenetV1/Logits/SpatialSqueeze_2reshape']
I     swap_reshape_softmax: remove node = ['MobilenetV1/Logits/SpatialSqueeze_2reshape', 'MobilenetV1/Predictions/Reshape_1'], add node = ['MobilenetV1/Predictions/Reshape_1', 'MobilenetV1/Logits/SpatialSqueeze_2reshape']
I     convert_avgpool_to_global: remove node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool'], add node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool_2global']
I     convert_softmax_to_exsoftmax13_equal: remove node = ['MobilenetV1/Predictions/Reshape_1'], add node = ['MobilenetV1/Predictions/Reshape_1']
I     convert_global_avgpool_to_conv: remove node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool_2global'], add node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool']
I     fold_constant ...
I     fold_constant done.
I fuse_ops done.
I 
W build: found outlier value, this may affect quantization accuracy
const name               abs_mean    abs_std     outlier value
const_fold_opt__306      3.48        7.25        42.282      
const_fold_opt__296      1.57        5.08        -52.270     
const_fold_opt__276      0.41        0.55        -10.869     
const_fold_opt__262      0.75        1.05        -25.778     
I sparse_weight ...
I sparse_weight done.
I 
Analysing : 100%|█████████████████████████████████████████████████| 58/58 [00:00<00:00, 4485.72it/s]
Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00, 225.70it/s]
I 
I fuse_ops ...
I fuse_ops with passes results:
I     fuse_two_dataconvert: remove node = ['MobilenetV1/Predictions/Reshape_1_int8__cvt_float16_int8', 'MobilenetV1/Predictions/Reshape_1__cvt_int8_to_float16'], add node = ['MobilenetV1/Predictions/Reshape_1_int8__cvt_float16_int8']
I     remove_invalid_dataconvert: remove node = ['MobilenetV1/Predictions/Reshape_1_int8__cvt_float16_int8']
I fuse_ops done.
I 
I quant_optimizer ...
I quant_optimizer results:
I     adjust_relu: ['Relu6__57', 'Relu6__55', 'Relu6__53', 'Relu6__51', 'Relu6__49', 'Relu6__47', 'Relu6__45', 'Relu6__43', 'Relu6__41', 'Relu6__39', 'Relu6__37', 'Relu6__35', 'Relu6__33', 'Relu6__31', 'Relu6__29', 'Relu6__27', 'Relu6__25', 'Relu6__23', 'Relu6__21', 'Relu6__19', 'Relu6__17', 'Relu6__15', 'Relu6__13', 'Relu6__11', 'Relu6__9', 'Relu6__7', 'Relu6__5']
I quant_optimizer done.
I 
I recover_const_share ...
I recover_const_share done.
I 
W build: The default input dtype of 'input' is changed from 'float32' to 'int8' in rknn model for performance!
                       Please take care of this change when deploy rknn model with Runtime API!
I rknn building ...
I RKNN: [15:13:15.626] compress = 0, conv_eltwise_activation_fuse = 1, global_fuse = 1, multi-core-model-mode = 7, output_optimize = 1,enable_argb_group=0
I RKNN: librknnc version: 1.5.2 (c6b7b351a@2023-08-23T15:34:44)
D RKNN: [15:13:15.651] RKNN is invoked
I RKNN: [15:13:15.707] Meet hybrid type, dtype: float16, tensor: MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__float16
I RKNN: [15:13:15.708] Meet hybrid type, dtype: float16, tensor: MobilenetV1/Predictions/Reshape_1
I RKNN: [15:13:15.708] Meet hybrid type, dtype: float16, tensor: MobilenetV1/Predictions/Reshape_1_before
D RKNN: [15:13:15.708] >>>>>> start: N4rknn19RKNNSetOpTargetPassE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn19RKNNSetOpTargetPassE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn16RKNNAddFirstConvE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn16RKNNAddFirstConvE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn27RKNNEliminateQATDataConvertE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn27RKNNEliminateQATDataConvertE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn17RKNNTileGroupConvE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn17RKNNTileGroupConvE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn19RKNNTileFcBatchFuseE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn19RKNNTileFcBatchFuseE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn15RKNNAddConvBiasE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn15RKNNAddConvBiasE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn15RKNNTileChannelE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn15RKNNTileChannelE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn18RKNNPerChannelPrepE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn18RKNNPerChannelPrepE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn11RKNNBnQuantE
D RKNN: [15:13:15.708] <<<<<<<< end: N4rknn11RKNNBnQuantE
D RKNN: [15:13:15.708] >>>>>> start: N4rknn21RKNNFuseOptimizerPassE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn21RKNNFuseOptimizerPassE
D RKNN: [15:13:15.709] >>>>>> start: N4rknn15RKNNTurnAutoPadE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn15RKNNTurnAutoPadE
D RKNN: [15:13:15.709] >>>>>> start: N4rknn16RKNNInitRNNConstE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn16RKNNInitRNNConstE
D RKNN: [15:13:15.709] >>>>>> start: N4rknn17RKNNInitCastConstE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn17RKNNInitCastConstE
D RKNN: [15:13:15.709] >>>>>> start: N4rknn20RKNNMultiSurfacePassE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn20RKNNMultiSurfacePassE
D RKNN: [15:13:15.709] >>>>>> start: OpEmit
D RKNN: [15:13:15.709] <<<<<<<< end: OpEmit
D RKNN: [15:13:15.709] >>>>>> start: N4rknn19RKNNLayoutMatchPassE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn19RKNNLayoutMatchPassE
D RKNN: [15:13:15.709] >>>>>> start: N4rknn20RKNNAddSecondaryNodeE
D RKNN: [15:13:15.709] <<<<<<<< end: N4rknn20RKNNAddSecondaryNodeE
D RKNN: [15:13:15.709] >>>>>> start: OpEmit
D RKNN: [15:13:15.710] not need tranpose
D RKNN: [15:13:15.710] not need tranpose
D RKNN: [15:13:15.710] finish initComputeZoneMap
D RKNN: [15:13:15.710] emit tp0
D RKNN: [15:13:15.710] emit max
D RKNN: [15:13:15.710] emit sub
D RKNN: [15:13:15.710] emit exp
D RKNN: [15:13:15.710] emit reduce sum
D RKNN: [15:13:15.711] emit prearare2
D RKNN: [15:13:15.711] emit div
D RKNN: [15:13:15.711] emit fpToint
D RKNN: [15:13:15.711] softmax fp16 out do not need fpToint
D RKNN: [15:13:15.711] emit tp1
D RKNN: [15:13:15.711] finish initComputeZoneMap
D RKNN: [15:13:15.711] <<<<<<<< end: OpEmit
D RKNN: [15:13:15.711] >>>>>> start: N4rknn23RKNNProfileAnalysisPassE
D RKNN: [15:13:15.711] node: Reshape:MobilenetV1/Logits/SpatialSqueeze_2reshape, Target: NPU
D RKNN: [15:13:15.711] <<<<<<<< end: N4rknn23RKNNProfileAnalysisPassE
D RKNN: [15:13:15.712] >>>>>> start: N4rknn21RKNNOperatorIdGenPassE
D RKNN: [15:13:15.712] <<<<<<<< end: N4rknn21RKNNOperatorIdGenPassE
D RKNN: [15:13:15.712] >>>>>> start: N4rknn23RKNNWeightTransposePassE
W RKNN: [15:13:15.899] Warning: Tensor MobilenetV1/Logits/SpatialSqueeze_2reshape_shape need paramter qtype, type is set to float16 by default!
W RKNN: [15:13:15.899] Warning: Tensor MobilenetV1/Logits/SpatialSqueeze_2reshape_shape need paramter qtype, type is set to float16 by default!
D RKNN: [15:13:15.900] <<<<<<<< end: N4rknn23RKNNWeightTransposePassE
D RKNN: [15:13:15.900] >>>>>> start: N4rknn26RKNNCPUWeightTransposePassE
D RKNN: [15:13:15.900] <<<<<<<< end: N4rknn26RKNNCPUWeightTransposePassE
D RKNN: [15:13:15.900] >>>>>> start: N4rknn18RKNNModelBuildPassE
D RKNN: [15:13:15.945] RKNNModelBuildPass: [Statistics]
D RKNN: [15:13:15.945] total_regcfg_size     :     48592
D RKNN: [15:13:15.945] total_diff_regcfg_size:     44720
D RKNN: [15:13:15.945] ID   OpType           DataType Target InputShape                                   OutputShape            DDR Cycles     NPU Cycles     Total Cycles   Time(us)       MacUsage(%)    Task Number    Lut Number     RW(KB)         FullName        
D RKNN: [15:13:15.945] 0    InputOperator    INT8     CPU    \                                            (1,3,224,224)          0              0              0              0              \              0              0              147.00         InputOperator:input
D RKNN: [15:13:15.945] 1    ConvClip         INT8     NPU    (1,3,224,224),(32,3,3,3),(32)                (1,32,112,112)         0              0              0              0              \              0              0              540.38         Conv:MobilenetV1/MobilenetV1/Conv2d_0/Relu6
D RKNN: [15:13:15.945] 2    ConvClip         INT8     NPU    (1,32,112,112),(1,32,3,3),(32)               (1,32,112,112)         0              0              0              0              \              0              0              784.47         Conv:MobilenetV1/MobilenetV1/Conv2d_1_depthwise/Relu6
D RKNN: [15:13:15.945] 3    ConvClip         INT8     NPU    (1,32,112,112),(64,32,1,1),(64)              (1,64,112,112)         0              0              0              0              \              0              0              1178.50        Conv:MobilenetV1/MobilenetV1/Conv2d_1_pointwise/Relu6
D RKNN: [15:13:15.945] 4    ConvClip         INT8     NPU    (1,64,112,112),(1,64,3,3),(64)               (1,64,56,56)           0              0              0              0              \              0              0              980.94         Conv:MobilenetV1/MobilenetV1/Conv2d_2_depthwise/Relu6
D RKNN: [15:13:15.945] 5    ConvClip         INT8     NPU    (1,64,56,56),(128,64,1,1),(128)              (1,128,56,56)          0              0              0              0              \              0              0              597.00         Conv:MobilenetV1/MobilenetV1/Conv2d_2_pointwise/Relu6
D RKNN: [15:13:15.945] 6    ConvClip         INT8     NPU    (1,128,56,56),(1,128,3,3),(128)              (1,128,56,56)          0              0              0              0              \              0              0              785.88         Conv:MobilenetV1/MobilenetV1/Conv2d_3_depthwise/Relu6
D RKNN: [15:13:15.945] 7    ConvClip         INT8     NPU    (1,128,56,56),(128,128,1,1),(128)            (1,128,56,56)          0              0              0              0              \              0              0              801.00         Conv:MobilenetV1/MobilenetV1/Conv2d_3_pointwise/Relu6
D RKNN: [15:13:15.945] 8    ConvClip         INT8     NPU    (1,128,56,56),(1,128,3,3),(128)              (1,128,28,28)          0              0              0              0              \              0              0              491.88         Conv:MobilenetV1/MobilenetV1/Conv2d_4_depthwise/Relu6
D RKNN: [15:13:15.945] 9    ConvClip         INT8     NPU    (1,128,28,28),(256,128,1,1),(256)            (1,256,28,28)          0              0              0              0              \              0              0              328.00         Conv:MobilenetV1/MobilenetV1/Conv2d_4_pointwise/Relu6
D RKNN: [15:13:15.945] 10   ConvClip         INT8     NPU    (1,256,28,28),(1,256,3,3),(256)              (1,256,28,28)          0              0              0              0              \              0              0              395.75         Conv:MobilenetV1/MobilenetV1/Conv2d_5_depthwise/Relu6
D RKNN: [15:13:15.945] 11   ConvClip         INT8     NPU    (1,256,28,28),(256,256,1,1),(256)            (1,256,28,28)          0              0              0              0              \              0              0              458.00         Conv:MobilenetV1/MobilenetV1/Conv2d_5_pointwise/Relu6
D RKNN: [15:13:15.945] 12   ConvClip         INT8     NPU    (1,256,28,28),(1,256,3,3),(256)              (1,256,14,14)          0              0              0              0              \              0              0              248.75         Conv:MobilenetV1/MobilenetV1/Conv2d_6_depthwise/Relu6
D RKNN: [15:13:15.945] 13   ConvClip         INT8     NPU    (1,256,14,14),(512,256,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              279.00         Conv:MobilenetV1/MobilenetV1/Conv2d_6_pointwise/Relu6
D RKNN: [15:13:15.945] 14   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_7_depthwise/Relu6
D RKNN: [15:13:15.945] 15   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_7_pointwise/Relu6
D RKNN: [15:13:15.945] 16   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_8_depthwise/Relu6
D RKNN: [15:13:15.945] 17   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_8_pointwise/Relu6
D RKNN: [15:13:15.945] 18   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_9_depthwise/Relu6
D RKNN: [15:13:15.945] 19   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_9_pointwise/Relu6
D RKNN: [15:13:15.945] 20   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_10_depthwise/Relu6
D RKNN: [15:13:15.945] 21   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_10_pointwise/Relu6
D RKNN: [15:13:15.945] 22   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_11_depthwise/Relu6
D RKNN: [15:13:15.945] 23   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_11_pointwise/Relu6
D RKNN: [15:13:15.945] 24   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,7,7)            0              0              0              0              \              0              0              130.00         Conv:MobilenetV1/MobilenetV1/Conv2d_12_depthwise/Relu6
D RKNN: [15:13:15.945] 25   ConvClip         INT8     NPU    (1,512,7,7),(1024,512,1,1),(1024)            (1,1024,7,7)           0              0              0              0              \              0              0              593.50         Conv:MobilenetV1/MobilenetV1/Conv2d_12_pointwise/Relu6
D RKNN: [15:13:15.945] 26   ConvClip         INT8     NPU    (1,1024,7,7),(1,1024,3,3),(1024)             (1,1024,7,7)           0              0              0              0              \              0              0              113.00         Conv:MobilenetV1/MobilenetV1/Conv2d_13_depthwise/Relu6
D RKNN: [15:13:15.945] 27   ConvClip         INT8     NPU    (1,1024,7,7),(1024,1024,1,1),(1024)          (1,1024,7,7)           0              0              0              0              \              0              0              1130.00        Conv:MobilenetV1/MobilenetV1/Conv2d_13_pointwise/Relu6
D RKNN: [15:13:15.945] 28   Conv             INT8     NPU    (1,1024,7,7),(1,1024,7,7),(1024)             (1,1024,1,1)           0              0              0              0              \              0              0              105.00         Conv:MobilenetV1/Logits/AvgPool_1a/AvgPool
D RKNN: [15:13:15.945] 29   Conv             INT8     NPU    (1,1024,1,1),(1001,1024,1,1),(1001)          (1,1001,1,1)           0              0              0              0              \              0              0              1010.86        Conv:MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd
D RKNN: [15:13:15.945] 30   exDataConvert    INT8     NPU    (1,1001,1,1)                                 (1,1001,1,1)           0              0              0              0              \              0              0              2.95           exDataConvert:MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__cvt_int8_float16
D RKNN: [15:13:15.945] 31   exSoftmax13      FLOAT16  NPU    (1,1001,1,1),(16,1008,1,1)                   (1,1001,1,1)           0              0              0              0              \              0              0              35.44          exSoftmax13:MobilenetV1/Predictions/Reshape_1
D RKNN: [15:13:15.945] 32   Reshape          FLOAT16  NPU    (1,1001,1,1),(2)                             (1,1001)               0              0              0              0              \              0              0              3.94           Reshape:MobilenetV1/Logits/SpatialSqueeze_2reshape
D RKNN: [15:13:15.945] 33   OutputOperator   FLOAT16  NPU    (1,1001),(1,1,1,1008)                        \                      0              0              0              0              \              0              0              17.71          OutputOperator:MobilenetV1/Predictions/Reshape_1
D RKNN: [15:13:15.945] <<<<<<<< end: N4rknn18RKNNModelBuildPassE
D RKNN: [15:13:15.945] >>>>>> start: N4rknn24RKNNModelRegCmdbuildPassE
D RKNN: [15:13:15.945] <<<<<<<< end: N4rknn24RKNNModelRegCmdbuildPassE
D RKNN: [15:13:15.945] >>>>>> start: N4rknn22RKNNMiniModelBuildPassE
D RKNN: [15:13:15.952] Export Mini RKNN model to /tmp/tmp9963wjbg/dumps/tf2onnx.mini.rknn
D RKNN: [15:13:15.952] <<<<<<<< end: N4rknn22RKNNMiniModelBuildPassE
D RKNN: [15:13:15.952] >>>>>> start: N4rknn21RKNNMemStatisticsPassE
D RKNN: [15:13:15.953] ---------------------------------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:13:15.953] ID  User           Tensor                                                        DataType  OrigShape      NativeShape      |     [Start       End)       Size
D RKNN: [15:13:15.953] ---------------------------------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:13:15.953] 1   ConvClip       input                                                         INT8      (1,3,224,224)  (1,1,224,224,3)  | 0x00454680 0x00479280 0x00024c00
D RKNN: [15:13:15.953] 2   ConvClip       Relu6__5:0                                                    INT8      (1,32,112,112) (1,2,112,112,16) | 0x00479280 0x004db280 0x00062000
D RKNN: [15:13:15.953] 3   ConvClip       Relu6__7:0                                                    INT8      (1,32,112,112) (1,2,112,112,16) | 0x004db280 0x0053d280 0x00062000
D RKNN: [15:13:15.953] 4   ConvClip       Relu6__9:0                                                    INT8      (1,64,112,112) (1,4,112,112,16) | 0x0053d280 0x00601280 0x000c4000
D RKNN: [15:13:15.953] 5   ConvClip       Relu6__11:0                                                   INT8      (1,64,56,56)   (1,5,56,56,16)   | 0x00454680 0x00491a80 0x0003d400
D RKNN: [15:13:15.953] 6   ConvClip       Relu6__13:0                                                   INT8      (1,128,56,56)  (1,8,56,56,16)   | 0x00491a80 0x004f3a80 0x00062000
D RKNN: [15:13:15.953] 7   ConvClip       Relu6__15:0                                                   INT8      (1,128,56,56)  (1,9,56,56,16)   | 0x004f3a80 0x00561e80 0x0006e400
D RKNN: [15:13:15.953] 8   ConvClip       Relu6__17:0                                                   INT8      (1,128,56,56)  (1,8,56,56,16)   | 0x00454680 0x004b6680 0x00062000
D RKNN: [15:13:15.953] 9   ConvClip       Relu6__19:0                                                   INT8      (1,128,28,28)  (1,9,28,28,16)   | 0x004b6680 0x004d1f80 0x0001b900
D RKNN: [15:13:15.953] 10  ConvClip       Relu6__21:0                                                   INT8      (1,256,28,28)  (1,16,28,28,16)  | 0x00454680 0x00485680 0x00031000
D RKNN: [15:13:15.953] 11  ConvClip       Relu6__23:0                                                   INT8      (1,256,28,28)  (1,16,28,28,16)  | 0x00485680 0x004b6680 0x00031000
D RKNN: [15:13:15.953] 12  ConvClip       Relu6__25:0                                                   INT8      (1,256,28,28)  (1,16,28,28,16)  | 0x00454680 0x00485680 0x00031000
D RKNN: [15:13:15.953] 13  ConvClip       Relu6__27:0                                                   INT8      (1,256,14,14)  (1,16,14,14,16)  | 0x00485680 0x00491a80 0x0000c400
D RKNN: [15:13:15.953] 14  ConvClip       Relu6__29:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:13:15.953] 15  ConvClip       Relu6__31:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:13:15.953] 16  ConvClip       Relu6__33:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:13:15.953] 17  ConvClip       Relu6__35:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:13:15.953] 18  ConvClip       Relu6__37:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:13:15.953] 19  ConvClip       Relu6__39:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:13:15.953] 20  ConvClip       Relu6__41:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:13:15.953] 21  ConvClip       Relu6__43:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:13:15.953] 22  ConvClip       Relu6__45:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:13:15.953] 23  ConvClip       Relu6__47:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:13:15.953] 24  ConvClip       Relu6__49:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:13:15.953] 25  ConvClip       Relu6__51:0                                                   INT8      (1,512,7,7)    (1,33,7,7,16)    | 0x0046ce80 0x00473680 0x00006800
D RKNN: [15:13:15.953] 26  ConvClip       Relu6__53:0                                                   INT8      (1,1024,7,7)   (1,67,7,7,16)    | 0x00454680 0x00461680 0x0000d000
D RKNN: [15:13:15.953] 27  ConvClip       Relu6__55:0                                                   INT8      (1,1024,7,7)   (1,67,7,7,16)    | 0x00461680 0x0046e680 0x0000d000
D RKNN: [15:13:15.953] 28  Conv           Relu6__57:0                                                   INT8      (1,1024,7,7)   (1,67,7,7,16)    | 0x00454680 0x00461680 0x0000d000
D RKNN: [15:13:15.953] 29  Conv           MobilenetV1/Logits/AvgPool_1a/AvgPool                         INT8      (1,1024,1,1)   (1,64,1,1,16)    | 0x00461680 0x00461a80 0x00000400
D RKNN: [15:13:15.953] 30  exDataConvert  MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd                      INT8      (1,1001,1,1)   (1,63,1,1,16)    | 0x00454680 0x00454a70 0x000003f0
D RKNN: [15:13:15.953] 31  exSoftmax13    MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__float16             FLOAT16   (1,1001,1,1)   (1,126,1,1,8)    | 0x00454a80 0x00455260 0x000007e0
D RKNN: [15:13:15.953] 31  exSoftmax13    MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__float16_exSecondary FLOAT16   (1,1001,1,1)   (1,1523,1,1,8)   | 0x00455280 0x0045b1b0 0x00005f30
D RKNN: [15:13:15.953] 32  Reshape        MobilenetV1/Predictions/Reshape_1_before                      FLOAT16   (1,1001,1,1)   (1,1651,1,1,8)   | 0x0045b1c0 0x004618f0 0x00006730
D RKNN: [15:13:15.953] 33  OutputOperator MobilenetV1/Predictions/Reshape_1                             FLOAT16   (1,1001)       (1,1001)         | 0x004546c0 0x00454ec0 0x00000800
D RKNN: [15:13:15.953] 33  OutputOperator MobilenetV1/Predictions/Reshape_1_exSecondary0                FLOAT16   (1,1,1,1008)   (1,0,1,1008,8)   | 0x00454ec0 0x004556a0 0x000007e0
D RKNN: [15:13:15.953] 33  OutputOperator MobilenetV1/Predictions/Reshape_1_exSecondary                 FLOAT16   (1,1,1,1001)   (1,1,1,1001)     | 0x004556c0 0x00455e92 0x000007d2
D RKNN: [15:13:15.953] ---------------------------------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:13:15.953] ---------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:13:15.953] ID  User     Tensor                                                      DataType  OrigShape       |     [Start       End)       Size
D RKNN: [15:13:15.953] ---------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:13:15.953] 1   ConvClip const_fold_opt__281                                         INT8      (32,3,3,3)      | 0x001ddec0 0x001de340 0x00000480
D RKNN: [15:13:15.953] 1   ConvClip MobilenetV1/MobilenetV1/Conv2d_0/Conv2D_bias                INT32     (32)            | 0x00421b80 0x00421c80 0x00000100
D RKNN: [15:13:15.953] 2   ConvClip const_fold_opt__306                                         INT8      (1,32,3,3)      | 0x00000000 0x00000240 0x00000240
D RKNN: [15:13:15.953] 2   ConvClip MobilenetV1/MobilenetV1/Conv2d_1_depthwise/depthwise_bias   INT32     (32)            | 0x00417ec0 0x00417f80 0x000000c0
D RKNN: [15:13:15.953] 3   ConvClip const_fold_opt__257                                         INT8      (64,32,1,1)     | 0x003ac140 0x003ac940 0x00000800
D RKNN: [15:13:15.953] 3   ConvClip MobilenetV1/MobilenetV1/Conv2d_1_pointwise/Conv2D_bias      INT32     (64)            | 0x00417cc0 0x00417ec0 0x00000200
D RKNN: [15:13:15.953] 4   ConvClip const_fold_opt__296                                         INT8      (1,64,3,3)      | 0x0004dc40 0x0004e0c0 0x00000480
D RKNN: [15:13:15.953] 4   ConvClip MobilenetV1/MobilenetV1/Conv2d_2_depthwise/depthwise_bias   INT32     (64)            | 0x00417b40 0x00417cc0 0x00000180
D RKNN: [15:13:15.953] 5   ConvClip const_fold_opt__280                                         INT8      (128,64,1,1)    | 0x001de340 0x001e0340 0x00002000
D RKNN: [15:13:15.953] 5   ConvClip MobilenetV1/MobilenetV1/Conv2d_2_pointwise/Conv2D_bias      INT32     (128)           | 0x00417740 0x00417b40 0x00000400
D RKNN: [15:13:15.953] 6   ConvClip const_fold_opt__276                                         INT8      (1,128,3,3)     | 0x001e0340 0x001e0c40 0x00000900
D RKNN: [15:13:15.953] 6   ConvClip MobilenetV1/MobilenetV1/Conv2d_3_depthwise/depthwise_bias   INT32     (128)           | 0x00417440 0x00417740 0x00000300
D RKNN: [15:13:15.953] 7   ConvClip const_fold_opt__273                                         INT8      (128,128,1,1)   | 0x001e0c40 0x001e4c40 0x00004000
D RKNN: [15:13:15.953] 7   ConvClip MobilenetV1/MobilenetV1/Conv2d_3_pointwise/Conv2D_bias      INT32     (128)           | 0x00417040 0x00417440 0x00000400
D RKNN: [15:13:15.953] 8   ConvClip const_fold_opt__268                                         INT8      (1,128,3,3)     | 0x00224c40 0x00225540 0x00000900
D RKNN: [15:13:15.953] 8   ConvClip MobilenetV1/MobilenetV1/Conv2d_4_depthwise/depthwise_bias   INT32     (128)           | 0x00416d40 0x00417040 0x00000300
D RKNN: [15:13:15.953] 9   ConvClip const_fold_opt__300                                         INT8      (256,128,1,1)   | 0x00004a40 0x0000ca40 0x00008000
D RKNN: [15:13:15.953] 9   ConvClip MobilenetV1/MobilenetV1/Conv2d_4_pointwise/Conv2D_bias      INT32     (256)           | 0x00416540 0x00416d40 0x00000800
D RKNN: [15:13:15.953] 10  ConvClip const_fold_opt__299                                         INT8      (1,256,3,3)     | 0x0000ca40 0x0000dc40 0x00001200
D RKNN: [15:13:15.953] 10  ConvClip MobilenetV1/MobilenetV1/Conv2d_5_depthwise/depthwise_bias   INT32     (256)           | 0x00415f40 0x00416540 0x00000600
D RKNN: [15:13:15.953] 11  ConvClip const_fold_opt__294                                         INT8      (256,256,1,1)   | 0x001484c0 0x001584c0 0x00010000
D RKNN: [15:13:15.953] 11  ConvClip MobilenetV1/MobilenetV1/Conv2d_5_pointwise/Conv2D_bias      INT32     (256)           | 0x00415740 0x00415f40 0x00000800
D RKNN: [15:13:15.953] 12  ConvClip const_fold_opt__293                                         INT8      (1,256,3,3)     | 0x001584c0 0x001596c0 0x00001200
D RKNN: [15:13:15.953] 12  ConvClip MobilenetV1/MobilenetV1/Conv2d_6_depthwise/depthwise_bias   INT32     (256)           | 0x00415140 0x00415740 0x00000600
D RKNN: [15:13:15.953] 13  ConvClip const_fold_opt__246                                         INT8      (512,256,1,1)   | 0x003aed40 0x003ced40 0x00020000
D RKNN: [15:13:15.953] 13  ConvClip MobilenetV1/MobilenetV1/Conv2d_6_pointwise/Conv2D_bias      INT32     (512)           | 0x00414140 0x00415140 0x00001000
D RKNN: [15:13:15.953] 14  ConvClip const_fold_opt__290                                         INT8      (1,512,3,3)     | 0x001596c0 0x0015bac0 0x00002400
D RKNN: [15:13:15.953] 14  ConvClip MobilenetV1/MobilenetV1/Conv2d_7_depthwise/depthwise_bias   INT32     (512)           | 0x00413540 0x00414140 0x00000c00
D RKNN: [15:13:15.953] 15  ConvClip const_fold_opt__288                                         INT8      (512,512,1,1)   | 0x0015bac0 0x0019bac0 0x00040000
D RKNN: [15:13:15.953] 15  ConvClip MobilenetV1/MobilenetV1/Conv2d_7_pointwise/Conv2D_bias      INT32     (512)           | 0x00412540 0x00413540 0x00001000
D RKNN: [15:13:15.953] 16  ConvClip const_fold_opt__286                                         INT8      (1,512,3,3)     | 0x0019bac0 0x0019dec0 0x00002400
D RKNN: [15:13:15.953] 16  ConvClip MobilenetV1/MobilenetV1/Conv2d_8_depthwise/depthwise_bias   INT32     (512)           | 0x00411940 0x00412540 0x00000c00
D RKNN: [15:13:15.953] 17  ConvClip const_fold_opt__284                                         INT8      (512,512,1,1)   | 0x0019dec0 0x001ddec0 0x00040000
D RKNN: [15:13:15.953] 17  ConvClip MobilenetV1/MobilenetV1/Conv2d_8_pointwise/Conv2D_bias      INT32     (512)           | 0x00410940 0x00411940 0x00001000
D RKNN: [15:13:15.953] 18  ConvClip const_fold_opt__254                                         INT8      (1,512,3,3)     | 0x003ac940 0x003aed40 0x00002400
D RKNN: [15:13:15.953] 18  ConvClip MobilenetV1/MobilenetV1/Conv2d_9_depthwise/depthwise_bias   INT32     (512)           | 0x0040fd40 0x00410940 0x00000c00
D RKNN: [15:13:15.953] 19  ConvClip const_fold_opt__297                                         INT8      (512,512,1,1)   | 0x0000dc40 0x0004dc40 0x00040000
D RKNN: [15:13:15.953] 19  ConvClip MobilenetV1/MobilenetV1/Conv2d_9_pointwise/Conv2D_bias      INT32     (512)           | 0x0040ed40 0x0040fd40 0x00001000
D RKNN: [15:13:15.953] 20  ConvClip const_fold_opt__301                                         INT8      (1,512,3,3)     | 0x00002640 0x00004a40 0x00002400
D RKNN: [15:13:15.953] 20  ConvClip MobilenetV1/MobilenetV1/Conv2d_10_depthwise/depthwise_bias  INT32     (512)           | 0x00420f80 0x00421b80 0x00000c00
D RKNN: [15:13:15.953] 21  ConvClip const_fold_opt__272                                         INT8      (512,512,1,1)   | 0x001e4c40 0x00224c40 0x00040000
D RKNN: [15:13:15.953] 21  ConvClip MobilenetV1/MobilenetV1/Conv2d_10_pointwise/Conv2D_bias     INT32     (512)           | 0x0041ff80 0x00420f80 0x00001000
D RKNN: [15:13:15.953] 22  ConvClip const_fold_opt__267                                         INT8      (1,512,3,3)     | 0x00225540 0x00227940 0x00002400
D RKNN: [15:13:15.953] 22  ConvClip MobilenetV1/MobilenetV1/Conv2d_11_depthwise/depthwise_bias  INT32     (512)           | 0x0041f380 0x0041ff80 0x00000c00
D RKNN: [15:13:15.953] 23  ConvClip const_fold_opt__242                                         INT8      (512,512,1,1)   | 0x003ced40 0x0040ed40 0x00040000
D RKNN: [15:13:15.953] 23  ConvClip MobilenetV1/MobilenetV1/Conv2d_11_pointwise/Conv2D_bias     INT32     (512)           | 0x0041e380 0x0041f380 0x00001000
D RKNN: [15:13:15.953] 24  ConvClip const_fold_opt__304                                         INT8      (1,512,3,3)     | 0x00000240 0x00002640 0x00002400
D RKNN: [15:13:15.953] 24  ConvClip MobilenetV1/MobilenetV1/Conv2d_12_depthwise/depthwise_bias  INT32     (512)           | 0x0041d780 0x0041e380 0x00000c00
D RKNN: [15:13:15.953] 25  ConvClip const_fold_opt__264                                         INT8      (1024,512,1,1)  | 0x00227940 0x002a7940 0x00080000
D RKNN: [15:13:15.953] 25  ConvClip MobilenetV1/MobilenetV1/Conv2d_12_pointwise/Conv2D_bias     INT32     (1024)          | 0x0041b780 0x0041d780 0x00002000
D RKNN: [15:13:15.953] 26  ConvClip const_fold_opt__262                                         INT8      (1,1024,3,3)    | 0x002a7940 0x002ac140 0x00004800
D RKNN: [15:13:15.953] 26  ConvClip MobilenetV1/MobilenetV1/Conv2d_13_depthwise/depthwise_bias  INT32     (1024)          | 0x00419f80 0x0041b780 0x00001800
D RKNN: [15:13:15.953] 27  ConvClip const_fold_opt__260                                         INT8      (1024,1024,1,1) | 0x002ac140 0x003ac140 0x00100000
D RKNN: [15:13:15.953] 27  ConvClip MobilenetV1/MobilenetV1/Conv2d_13_pointwise/Conv2D_bias     INT32     (1024)          | 0x00417f80 0x00419f80 0x00002000
D RKNN: [15:13:15.953] 28  Conv     MobilenetV1/Logits/AvgPool_1a/AvgPool_2global_2conv_weight0 INT8      (1,1024,7,7)    | 0x00423c40 0x0043c440 0x00018800
D RKNN: [15:13:15.953] 28  Conv     MobilenetV1/Logits/AvgPool_1a/AvgPool_2global_2conv_bias0   INT32     (1024)          | 0x0043c440*0x0043dc40 0x00001800
D RKNN: [15:13:15.953] 29  Conv     const_fold_opt__295                                         INT8      (1001,1024,1,1) | 0x0004e0c0 0x001484c0 0x000fa400
D RKNN: [15:13:15.953] 29  Conv     MobilenetV1/Logits/Conv2d_1c_1x1/Conv2D_bias                INT32     (1001)          | 0x00421c80 0x00423c00 0x00001f80
D RKNN: [15:13:15.953] 32  Reshape  MobilenetV1/Logits/SpatialSqueeze_2reshape_shape            INT64     (2)             | 0x00423c00 0x00423c40 0x00000040
D RKNN: [15:13:15.953] ---------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:13:15.953] ----------------------------------------
D RKNN: [15:13:15.953] Total Weight Memory Size: 4447296
D RKNN: [15:13:15.953] Total Internal Memory Size: 1756160
D RKNN: [15:13:15.953] Predict Internal Memory RW Amount: 10570545
D RKNN: [15:13:15.953] Predict Weight Memory RW Amount: 4479552
D RKNN: [15:13:15.953] ----------------------------------------
D RKNN: [15:13:15.953] <<<<<<<< end: N4rknn21RKNNMemStatisticsPassE
I rknn buiding done.
done
--> Export rknn model
done
--> Init runtime environment
W init_runtime: Target is None, use simulator!
done
--> Running model
W inference: The 'data_format' has not been set and defaults is nhwc!
Analysing : 100%|█████████████████████████████████████████████████| 60/60 [00:00<00:00, 4336.02it/s]
Preparing : 100%|██████████████████████████████████████████████████| 60/60 [00:00<00:00, 529.39it/s]
mobilenet_v1
-----TOP 5-----
[156]: 0.92822265625
[155]: 0.06317138671875
[205]: 0.004299163818359375
[284]: 0.0030956268310546875
[285]: 0.00017058849334716797

done
```

#### Running on CT36L/CT36B NPU platform connected to PC

RKNN-Toolkit2 connects to the OTG device CT36L/CT36B via PC's USB. RKNN-Toolkit2
Transfer the RKNN model to the NPU of CT36L/CT36B to run, and then obtain inference results, performance information, etc. from CT36L/CT36B:


* First deploy the CT36L/CT36B environment: update librknnrt.so and run rknn_server
* *Linux*

```shell
adb push rknn-toolkit2/rknpu2/runtime/Linux/rknn_server/armhf/usr/bin/rknn_server /userdata/
adb push rknn-toolkit2/rknpu2/runtime/Linux/librknn_api/armhf/librknnrt.so /userdata/

# Declare the library file path that rknn_server depends on. This step is very important, not declaring the library file path will cause a segfault! ! !
export LD_LIBRARY_PATH=/userdata/

# Please run rknn_server in the serial port terminal of the board
chmod +x /userdata/rknn_server
/userdata/rknn_server
```

* Then modify the `examples/tflite/mobilenet_v1/test.py` file on the PC and add the target platform to it.

```shell
diff --git a/examples/tflite/mobilenet_v1/test.py b/examples/tflite/mobilenet_v1/test.py
index cc8d3f9..f6e28fd 100755
--- a/examples/tflite/mobilenet_v1/test.py
+++ b/examples/tflite/mobilenet_v1/test.py
@@ -28,7 +28,7 @@ if __name__ == '__main__':
 
     # Pre-process config
     print('--> Config model')
-    rknn.config(mean_values=[128, 128, 128], std_values=[128, 128, 128], target_platform='rk3566')
+    rknn.config(mean_values=[128, 128, 128], std_values=[128, 128, 128], target_platform='rv1106')
     print('done')
 
     # Load model (from https://www.tensorflow.org/lite/guide/hosted_models?hl=zh-cn)
@@ -62,7 +62,7 @@ if __name__ == '__main__':
 
     # Init runtime environment
     print('--> Init runtime environment')
-    ret = rknn.init_runtime()
+    ret = rknn.init_runtime(target='rv1106')
     if ret != 0:
         print('Init runtime environment failed!')
         exit(ret)

```
* Run test.py on PC
```shell
(.venv) firefly@firefly:~/rv1106/rknn-toolkit2/examples/tflite/mobilenet_v1$ python3 test.py
W __init__: rknn-toolkit2 version: 1.5.2+b642f30c
--> Config model
done
--> Loading model
2023-12-22 15:05:39.125720: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcudart.so.11.0'; dlerror: libcudart.so.11.0: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: /home/lvsx/.venv/lib/python3.8/site-packages/cv2/../../lib64:
2023-12-22 15:05:39.125744: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
INFO: Created TensorFlow Lite XNNPACK delegate for CPU.
Loading : 100%|██████████████████████████████████████████████████| 58/58 [00:00<00:00, 17265.41it/s]
done
--> Building model
I base_optimize ...
I base_optimize done.
I 
I fold_constant ...
I fold_constant done.
I 
I correct_ops ...
I correct_ops done.
I 
I fuse_ops ...
I fuse_ops results:
I     convert_squeeze_to_reshape: remove node = ['MobilenetV1/Logits/SpatialSqueeze'], add node = ['MobilenetV1/Logits/SpatialSqueeze_2reshape']
I     swap_reshape_softmax: remove node = ['MobilenetV1/Logits/SpatialSqueeze_2reshape', 'MobilenetV1/Predictions/Reshape_1'], add node = ['MobilenetV1/Predictions/Reshape_1', 'MobilenetV1/Logits/SpatialSqueeze_2reshape']
I     convert_avgpool_to_global: remove node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool'], add node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool_2global']
I     convert_softmax_to_exsoftmax13_equal: remove node = ['MobilenetV1/Predictions/Reshape_1'], add node = ['MobilenetV1/Predictions/Reshape_1']
I     convert_global_avgpool_to_conv: remove node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool_2global'], add node = ['MobilenetV1/Logits/AvgPool_1a/AvgPool']
I     fold_constant ...
I     fold_constant done.
I fuse_ops done.
I 
W build: found outlier value, this may affect quantization accuracy
const name               abs_mean    abs_std     outlier value
const_fold_opt__281      3.48        7.25        42.282      
const_fold_opt__275      1.57        5.08        -52.270     
const_fold_opt__296      0.41        0.55        -10.869     
const_fold_opt__285      0.75        1.05        -25.778     
I sparse_weight ...
I sparse_weight done.
I 
Analysing : 100%|█████████████████████████████████████████████████| 58/58 [00:00<00:00, 4367.58it/s]
Quantizating : 100%|███████████████████████████████████████████████| 58/58 [00:00<00:00, 455.04it/s]
I 
I fuse_ops ...
I fuse_ops with passes results:
I     fuse_two_dataconvert: remove node = ['MobilenetV1/Predictions/Reshape_1_int8__cvt_float16_int8', 'MobilenetV1/Predictions/Reshape_1__cvt_int8_to_float16'], add node = ['MobilenetV1/Predictions/Reshape_1_int8__cvt_float16_int8']
I     remove_invalid_dataconvert: remove node = ['MobilenetV1/Predictions/Reshape_1_int8__cvt_float16_int8']
I fuse_ops done.
I 
I quant_optimizer ...
I quant_optimizer results:
I     adjust_relu: ['Relu6__57', 'Relu6__55', 'Relu6__53', 'Relu6__51', 'Relu6__49', 'Relu6__47', 'Relu6__45', 'Relu6__43', 'Relu6__41', 'Relu6__39', 'Relu6__37', 'Relu6__35', 'Relu6__33', 'Relu6__31', 'Relu6__29', 'Relu6__27', 'Relu6__25', 'Relu6__23', 'Relu6__21', 'Relu6__19', 'Relu6__17', 'Relu6__15', 'Relu6__13', 'Relu6__11', 'Relu6__9', 'Relu6__7', 'Relu6__5']
I quant_optimizer done.
I 
I recover_const_share ...
I recover_const_share done.
I 
W build: The default input dtype of 'input' is changed from 'float32' to 'int8' in rknn model for performance!
                       Please take care of this change when deploy rknn model with Runtime API!
I rknn building ...
I RKNN: [15:05:44.321] compress = 0, conv_eltwise_activation_fuse = 1, global_fuse = 1, multi-core-model-mode = 7, output_optimize = 1,enable_argb_group=0
I RKNN: librknnc version: 1.5.2 (c6b7b351a@2023-08-23T15:34:44)
D RKNN: [15:05:44.346] RKNN is invoked
I RKNN: [15:05:44.401] Meet hybrid type, dtype: float16, tensor: MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__float16
I RKNN: [15:05:44.401] Meet hybrid type, dtype: float16, tensor: MobilenetV1/Predictions/Reshape_1
I RKNN: [15:05:44.401] Meet hybrid type, dtype: float16, tensor: MobilenetV1/Predictions/Reshape_1_before
D RKNN: [15:05:44.401] >>>>>> start: N4rknn19RKNNSetOpTargetPassE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn19RKNNSetOpTargetPassE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn16RKNNAddFirstConvE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn16RKNNAddFirstConvE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn27RKNNEliminateQATDataConvertE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn27RKNNEliminateQATDataConvertE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn17RKNNTileGroupConvE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn17RKNNTileGroupConvE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn19RKNNTileFcBatchFuseE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn19RKNNTileFcBatchFuseE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn15RKNNAddConvBiasE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn15RKNNAddConvBiasE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn15RKNNTileChannelE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn15RKNNTileChannelE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn18RKNNPerChannelPrepE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn18RKNNPerChannelPrepE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn11RKNNBnQuantE
D RKNN: [15:05:44.401] <<<<<<<< end: N4rknn11RKNNBnQuantE
D RKNN: [15:05:44.401] >>>>>> start: N4rknn21RKNNFuseOptimizerPassE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn21RKNNFuseOptimizerPassE
D RKNN: [15:05:44.402] >>>>>> start: N4rknn15RKNNTurnAutoPadE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn15RKNNTurnAutoPadE
D RKNN: [15:05:44.402] >>>>>> start: N4rknn16RKNNInitRNNConstE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn16RKNNInitRNNConstE
D RKNN: [15:05:44.402] >>>>>> start: N4rknn17RKNNInitCastConstE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn17RKNNInitCastConstE
D RKNN: [15:05:44.402] >>>>>> start: N4rknn20RKNNMultiSurfacePassE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn20RKNNMultiSurfacePassE
D RKNN: [15:05:44.402] >>>>>> start: OpEmit
D RKNN: [15:05:44.402] <<<<<<<< end: OpEmit
D RKNN: [15:05:44.402] >>>>>> start: N4rknn19RKNNLayoutMatchPassE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn19RKNNLayoutMatchPassE
D RKNN: [15:05:44.402] >>>>>> start: N4rknn20RKNNAddSecondaryNodeE
D RKNN: [15:05:44.402] <<<<<<<< end: N4rknn20RKNNAddSecondaryNodeE
D RKNN: [15:05:44.402] >>>>>> start: OpEmit
D RKNN: [15:05:44.403] not need tranpose
D RKNN: [15:05:44.403] not need tranpose
D RKNN: [15:05:44.403] finish initComputeZoneMap
D RKNN: [15:05:44.403] emit tp0
D RKNN: [15:05:44.403] emit max
D RKNN: [15:05:44.403] emit sub
D RKNN: [15:05:44.403] emit exp
D RKNN: [15:05:44.404] emit reduce sum
D RKNN: [15:05:44.404] emit prearare2
D RKNN: [15:05:44.404] emit div
D RKNN: [15:05:44.404] emit fpToint
D RKNN: [15:05:44.404] softmax fp16 out do not need fpToint
D RKNN: [15:05:44.404] emit tp1
D RKNN: [15:05:44.404] finish initComputeZoneMap
D RKNN: [15:05:44.404] <<<<<<<< end: OpEmit
D RKNN: [15:05:44.404] >>>>>> start: N4rknn23RKNNProfileAnalysisPassE
D RKNN: [15:05:44.404] node: Reshape:MobilenetV1/Logits/SpatialSqueeze_2reshape, Target: NPU
D RKNN: [15:05:44.404] <<<<<<<< end: N4rknn23RKNNProfileAnalysisPassE
D RKNN: [15:05:44.405] >>>>>> start: N4rknn21RKNNOperatorIdGenPassE
D RKNN: [15:05:44.405] <<<<<<<< end: N4rknn21RKNNOperatorIdGenPassE
D RKNN: [15:05:44.405] >>>>>> start: N4rknn23RKNNWeightTransposePassE
W RKNN: [15:05:44.592] Warning: Tensor MobilenetV1/Logits/SpatialSqueeze_2reshape_shape need paramter qtype, type is set to float16 by default!
W RKNN: [15:05:44.592] Warning: Tensor MobilenetV1/Logits/SpatialSqueeze_2reshape_shape need paramter qtype, type is set to float16 by default!
D RKNN: [15:05:44.593] <<<<<<<< end: N4rknn23RKNNWeightTransposePassE
D RKNN: [15:05:44.593] >>>>>> start: N4rknn26RKNNCPUWeightTransposePassE
D RKNN: [15:05:44.594] <<<<<<<< end: N4rknn26RKNNCPUWeightTransposePassE
D RKNN: [15:05:44.594] >>>>>> start: N4rknn18RKNNModelBuildPassE
D RKNN: [15:05:44.638] RKNNModelBuildPass: [Statistics]
D RKNN: [15:05:44.638] total_regcfg_size     :     48592
D RKNN: [15:05:44.638] total_diff_regcfg_size:     44720
D RKNN: [15:05:44.638] ID   OpType           DataType Target InputShape                                   OutputShape            DDR Cycles     NPU Cycles     Total Cycles   Time(us)       MacUsage(%)    Task Number    Lut Number     RW(KB)         FullName        
D RKNN: [15:05:44.638] 0    InputOperator    INT8     CPU    \                                            (1,3,224,224)          0              0              0              0              \              0              0              147.00         InputOperator:input
D RKNN: [15:05:44.638] 1    ConvClip         INT8     NPU    (1,3,224,224),(32,3,3,3),(32)                (1,32,112,112)         0              0              0              0              \              0              0              540.38         Conv:MobilenetV1/MobilenetV1/Conv2d_0/Relu6
D RKNN: [15:05:44.638] 2    ConvClip         INT8     NPU    (1,32,112,112),(1,32,3,3),(32)               (1,32,112,112)         0              0              0              0              \              0              0              784.47         Conv:MobilenetV1/MobilenetV1/Conv2d_1_depthwise/Relu6
D RKNN: [15:05:44.638] 3    ConvClip         INT8     NPU    (1,32,112,112),(64,32,1,1),(64)              (1,64,112,112)         0              0              0              0              \              0              0              1178.50        Conv:MobilenetV1/MobilenetV1/Conv2d_1_pointwise/Relu6
D RKNN: [15:05:44.638] 4    ConvClip         INT8     NPU    (1,64,112,112),(1,64,3,3),(64)               (1,64,56,56)           0              0              0              0              \              0              0              980.94         Conv:MobilenetV1/MobilenetV1/Conv2d_2_depthwise/Relu6
D RKNN: [15:05:44.638] 5    ConvClip         INT8     NPU    (1,64,56,56),(128,64,1,1),(128)              (1,128,56,56)          0              0              0              0              \              0              0              597.00         Conv:MobilenetV1/MobilenetV1/Conv2d_2_pointwise/Relu6
D RKNN: [15:05:44.638] 6    ConvClip         INT8     NPU    (1,128,56,56),(1,128,3,3),(128)              (1,128,56,56)          0              0              0              0              \              0              0              785.88         Conv:MobilenetV1/MobilenetV1/Conv2d_3_depthwise/Relu6
D RKNN: [15:05:44.638] 7    ConvClip         INT8     NPU    (1,128,56,56),(128,128,1,1),(128)            (1,128,56,56)          0              0              0              0              \              0              0              801.00         Conv:MobilenetV1/MobilenetV1/Conv2d_3_pointwise/Relu6
D RKNN: [15:05:44.638] 8    ConvClip         INT8     NPU    (1,128,56,56),(1,128,3,3),(128)              (1,128,28,28)          0              0              0              0              \              0              0              491.88         Conv:MobilenetV1/MobilenetV1/Conv2d_4_depthwise/Relu6
D RKNN: [15:05:44.638] 9    ConvClip         INT8     NPU    (1,128,28,28),(256,128,1,1),(256)            (1,256,28,28)          0              0              0              0              \              0              0              328.00         Conv:MobilenetV1/MobilenetV1/Conv2d_4_pointwise/Relu6
D RKNN: [15:05:44.638] 10   ConvClip         INT8     NPU    (1,256,28,28),(1,256,3,3),(256)              (1,256,28,28)          0              0              0              0              \              0              0              395.75         Conv:MobilenetV1/MobilenetV1/Conv2d_5_depthwise/Relu6
D RKNN: [15:05:44.638] 11   ConvClip         INT8     NPU    (1,256,28,28),(256,256,1,1),(256)            (1,256,28,28)          0              0              0              0              \              0              0              458.00         Conv:MobilenetV1/MobilenetV1/Conv2d_5_pointwise/Relu6
D RKNN: [15:05:44.638] 12   ConvClip         INT8     NPU    (1,256,28,28),(1,256,3,3),(256)              (1,256,14,14)          0              0              0              0              \              0              0              248.75         Conv:MobilenetV1/MobilenetV1/Conv2d_6_depthwise/Relu6
D RKNN: [15:05:44.638] 13   ConvClip         INT8     NPU    (1,256,14,14),(512,256,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              279.00         Conv:MobilenetV1/MobilenetV1/Conv2d_6_pointwise/Relu6
D RKNN: [15:05:44.638] 14   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_7_depthwise/Relu6
D RKNN: [15:05:44.638] 15   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_7_pointwise/Relu6
D RKNN: [15:05:44.638] 16   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_8_depthwise/Relu6
D RKNN: [15:05:44.638] 17   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_8_pointwise/Relu6
D RKNN: [15:05:44.638] 18   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_9_depthwise/Relu6
D RKNN: [15:05:44.638] 19   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_9_pointwise/Relu6
D RKNN: [15:05:44.638] 20   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_10_depthwise/Relu6
D RKNN: [15:05:44.638] 21   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_10_pointwise/Relu6
D RKNN: [15:05:44.638] 22   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,14,14)          0              0              0              0              \              0              0              203.50         Conv:MobilenetV1/MobilenetV1/Conv2d_11_depthwise/Relu6
D RKNN: [15:05:44.638] 23   ConvClip         INT8     NPU    (1,512,14,14),(512,512,1,1),(512)            (1,512,14,14)          0              0              0              0              \              0              0              456.00         Conv:MobilenetV1/MobilenetV1/Conv2d_11_pointwise/Relu6
D RKNN: [15:05:44.638] 24   ConvClip         INT8     NPU    (1,512,14,14),(1,512,3,3),(512)              (1,512,7,7)            0              0              0              0              \              0              0              130.00         Conv:MobilenetV1/MobilenetV1/Conv2d_12_depthwise/Relu6
D RKNN: [15:05:44.638] 25   ConvClip         INT8     NPU    (1,512,7,7),(1024,512,1,1),(1024)            (1,1024,7,7)           0              0              0              0              \              0              0              593.50         Conv:MobilenetV1/MobilenetV1/Conv2d_12_pointwise/Relu6
D RKNN: [15:05:44.638] 26   ConvClip         INT8     NPU    (1,1024,7,7),(1,1024,3,3),(1024)             (1,1024,7,7)           0              0              0              0              \              0              0              113.00         Conv:MobilenetV1/MobilenetV1/Conv2d_13_depthwise/Relu6
D RKNN: [15:05:44.638] 27   ConvClip         INT8     NPU    (1,1024,7,7),(1024,1024,1,1),(1024)          (1,1024,7,7)           0              0              0              0              \              0              0              1130.00        Conv:MobilenetV1/MobilenetV1/Conv2d_13_pointwise/Relu6
D RKNN: [15:05:44.638] 28   Conv             INT8     NPU    (1,1024,7,7),(1,1024,7,7),(1024)             (1,1024,1,1)           0              0              0              0              \              0              0              105.00         Conv:MobilenetV1/Logits/AvgPool_1a/AvgPool
D RKNN: [15:05:44.638] 29   Conv             INT8     NPU    (1,1024,1,1),(1001,1024,1,1),(1001)          (1,1001,1,1)           0              0              0              0              \              0              0              1010.86        Conv:MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd
D RKNN: [15:05:44.638] 30   exDataConvert    INT8     NPU    (1,1001,1,1)                                 (1,1001,1,1)           0              0              0              0              \              0              0              2.95           exDataConvert:MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__cvt_int8_float16
D RKNN: [15:05:44.638] 31   exSoftmax13      FLOAT16  NPU    (1,1001,1,1),(16,1008,1,1)                   (1,1001,1,1)           0              0              0              0              \              0              0              35.44          exSoftmax13:MobilenetV1/Predictions/Reshape_1
D RKNN: [15:05:44.638] 32   Reshape          FLOAT16  NPU    (1,1001,1,1),(2)                             (1,1001)               0              0              0              0              \              0              0              3.94           Reshape:MobilenetV1/Logits/SpatialSqueeze_2reshape
D RKNN: [15:05:44.638] 33   OutputOperator   FLOAT16  NPU    (1,1001),(1,1,1,1008)                        \                      0              0              0              0              \              0              0              17.71          OutputOperator:MobilenetV1/Predictions/Reshape_1
D RKNN: [15:05:44.638] <<<<<<<< end: N4rknn18RKNNModelBuildPassE
D RKNN: [15:05:44.638] >>>>>> start: N4rknn24RKNNModelRegCmdbuildPassE
D RKNN: [15:05:44.638] <<<<<<<< end: N4rknn24RKNNModelRegCmdbuildPassE
D RKNN: [15:05:44.638] >>>>>> start: N4rknn22RKNNMiniModelBuildPassE
D RKNN: [15:05:44.644] Export Mini RKNN model to /tmp/tmpqauv0aou/dumps/tf2onnx.mini.rknn
D RKNN: [15:05:44.644] <<<<<<<< end: N4rknn22RKNNMiniModelBuildPassE
D RKNN: [15:05:44.645] >>>>>> start: N4rknn21RKNNMemStatisticsPassE
D RKNN: [15:05:44.645] ---------------------------------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:05:44.645] ID  User           Tensor                                                        DataType  OrigShape      NativeShape      |     [Start       End)       Size
D RKNN: [15:05:44.645] ---------------------------------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:05:44.645] 1   ConvClip       input                                                         INT8      (1,3,224,224)  (1,1,224,224,3)  | 0x00454680 0x00479280 0x00024c00
D RKNN: [15:05:44.645] 2   ConvClip       Relu6__5:0                                                    INT8      (1,32,112,112) (1,2,112,112,16) | 0x00479280 0x004db280 0x00062000
D RKNN: [15:05:44.645] 3   ConvClip       Relu6__7:0                                                    INT8      (1,32,112,112) (1,2,112,112,16) | 0x004db280 0x0053d280 0x00062000
D RKNN: [15:05:44.645] 4   ConvClip       Relu6__9:0                                                    INT8      (1,64,112,112) (1,4,112,112,16) | 0x0053d280 0x00601280 0x000c4000
D RKNN: [15:05:44.645] 5   ConvClip       Relu6__11:0                                                   INT8      (1,64,56,56)   (1,5,56,56,16)   | 0x00454680 0x00491a80 0x0003d400
D RKNN: [15:05:44.645] 6   ConvClip       Relu6__13:0                                                   INT8      (1,128,56,56)  (1,8,56,56,16)   | 0x00491a80 0x004f3a80 0x00062000
D RKNN: [15:05:44.645] 7   ConvClip       Relu6__15:0                                                   INT8      (1,128,56,56)  (1,9,56,56,16)   | 0x004f3a80 0x00561e80 0x0006e400
D RKNN: [15:05:44.645] 8   ConvClip       Relu6__17:0                                                   INT8      (1,128,56,56)  (1,8,56,56,16)   | 0x00454680 0x004b6680 0x00062000
D RKNN: [15:05:44.645] 9   ConvClip       Relu6__19:0                                                   INT8      (1,128,28,28)  (1,9,28,28,16)   | 0x004b6680 0x004d1f80 0x0001b900
D RKNN: [15:05:44.645] 10  ConvClip       Relu6__21:0                                                   INT8      (1,256,28,28)  (1,16,28,28,16)  | 0x00454680 0x00485680 0x00031000
D RKNN: [15:05:44.645] 11  ConvClip       Relu6__23:0                                                   INT8      (1,256,28,28)  (1,16,28,28,16)  | 0x00485680 0x004b6680 0x00031000
D RKNN: [15:05:44.645] 12  ConvClip       Relu6__25:0                                                   INT8      (1,256,28,28)  (1,16,28,28,16)  | 0x00454680 0x00485680 0x00031000
D RKNN: [15:05:44.645] 13  ConvClip       Relu6__27:0                                                   INT8      (1,256,14,14)  (1,16,14,14,16)  | 0x00485680 0x00491a80 0x0000c400
D RKNN: [15:05:44.645] 14  ConvClip       Relu6__29:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:05:44.645] 15  ConvClip       Relu6__31:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:05:44.645] 16  ConvClip       Relu6__33:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:05:44.645] 17  ConvClip       Relu6__35:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:05:44.645] 18  ConvClip       Relu6__37:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:05:44.645] 19  ConvClip       Relu6__39:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:05:44.645] 20  ConvClip       Relu6__41:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:05:44.645] 21  ConvClip       Relu6__43:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:05:44.645] 22  ConvClip       Relu6__45:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:05:44.645] 23  ConvClip       Relu6__47:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x0046ce80 0x00485680 0x00018800
D RKNN: [15:05:44.645] 24  ConvClip       Relu6__49:0                                                   INT8      (1,512,14,14)  (1,32,14,14,16)  | 0x00454680 0x0046ce80 0x00018800
D RKNN: [15:05:44.645] 25  ConvClip       Relu6__51:0                                                   INT8      (1,512,7,7)    (1,33,7,7,16)    | 0x0046ce80 0x00473680 0x00006800
D RKNN: [15:05:44.645] 26  ConvClip       Relu6__53:0                                                   INT8      (1,1024,7,7)   (1,67,7,7,16)    | 0x00454680 0x00461680 0x0000d000
D RKNN: [15:05:44.645] 27  ConvClip       Relu6__55:0                                                   INT8      (1,1024,7,7)   (1,67,7,7,16)    | 0x00461680 0x0046e680 0x0000d000
D RKNN: [15:05:44.645] 28  Conv           Relu6__57:0                                                   INT8      (1,1024,7,7)   (1,67,7,7,16)    | 0x00454680 0x00461680 0x0000d000
D RKNN: [15:05:44.645] 29  Conv           MobilenetV1/Logits/AvgPool_1a/AvgPool                         INT8      (1,1024,1,1)   (1,64,1,1,16)    | 0x00461680 0x00461a80 0x00000400
D RKNN: [15:05:44.645] 30  exDataConvert  MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd                      INT8      (1,1001,1,1)   (1,63,1,1,16)    | 0x00454680 0x00454a70 0x000003f0
D RKNN: [15:05:44.645] 31  exSoftmax13    MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__float16             FLOAT16   (1,1001,1,1)   (1,126,1,1,8)    | 0x00454a80 0x00455260 0x000007e0
D RKNN: [15:05:44.645] 31  exSoftmax13    MobilenetV1/Logits/Conv2d_1c_1x1/BiasAdd__float16_exSecondary FLOAT16   (1,1001,1,1)   (1,1523,1,1,8)   | 0x00455280 0x0045b1b0 0x00005f30
D RKNN: [15:05:44.645] 32  Reshape        MobilenetV1/Predictions/Reshape_1_before                      FLOAT16   (1,1001,1,1)   (1,1651,1,1,8)   | 0x0045b1c0 0x004618f0 0x00006730
D RKNN: [15:05:44.645] 33  OutputOperator MobilenetV1/Predictions/Reshape_1                             FLOAT16   (1,1001)       (1,1001)         | 0x004546c0 0x00454ec0 0x00000800
D RKNN: [15:05:44.645] 33  OutputOperator MobilenetV1/Predictions/Reshape_1_exSecondary0                FLOAT16   (1,1,1,1008)   (1,0,1,1008,8)   | 0x00454ec0 0x004556a0 0x000007e0
D RKNN: [15:05:44.645] 33  OutputOperator MobilenetV1/Predictions/Reshape_1_exSecondary                 FLOAT16   (1,1,1,1001)   (1,1,1,1001)     | 0x004556c0 0x00455e92 0x000007d2
D RKNN: [15:05:44.645] ---------------------------------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:05:44.645] ---------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:05:44.645] ID  User     Tensor                                                      DataType  OrigShape       |     [Start       End)       Size
D RKNN: [15:05:44.645] ---------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:05:44.645] 1   ConvClip const_fold_opt__248                                         INT8      (32,3,3,3)      | 0x002c68c0 0x002c6d40 0x00000480
D RKNN: [15:05:44.645] 1   ConvClip MobilenetV1/MobilenetV1/Conv2d_0/Conv2D_bias                INT32     (32)            | 0x00421b80 0x00421c80 0x00000100
D RKNN: [15:05:44.645] 2   ConvClip const_fold_opt__281                                         INT8      (1,32,3,3)      | 0x0001ac00 0x0001ae40 0x00000240
D RKNN: [15:05:44.645] 2   ConvClip MobilenetV1/MobilenetV1/Conv2d_1_depthwise/depthwise_bias   INT32     (32)            | 0x00417ec0 0x00417f80 0x000000c0
D RKNN: [15:05:44.645] 3   ConvClip const_fold_opt__303                                         INT8      (64,32,1,1)     | 0x00004400 0x00004c00 0x00000800
D RKNN: [15:05:44.645] 3   ConvClip MobilenetV1/MobilenetV1/Conv2d_1_pointwise/Conv2D_bias      INT32     (64)            | 0x00417cc0 0x00417ec0 0x00000200
D RKNN: [15:05:44.645] 4   ConvClip const_fold_opt__275                                         INT8      (1,64,3,3)      | 0x0002c040 0x0002c4c0 0x00000480
D RKNN: [15:05:44.645] 4   ConvClip MobilenetV1/MobilenetV1/Conv2d_2_depthwise/depthwise_bias   INT32     (64)            | 0x00417b40 0x00417cc0 0x00000180
D RKNN: [15:05:44.645] 5   ConvClip const_fold_opt__307                                         INT8      (128,64,1,1)    | 0x00000000 0x00002000 0x00002000
D RKNN: [15:05:44.645] 5   ConvClip MobilenetV1/MobilenetV1/Conv2d_2_pointwise/Conv2D_bias      INT32     (128)           | 0x00417740 0x00417b40 0x00000400
D RKNN: [15:05:44.645] 6   ConvClip const_fold_opt__296                                         INT8      (1,128,3,3)     | 0x00008200 0x00008b00 0x00000900
D RKNN: [15:05:44.645] 6   ConvClip MobilenetV1/MobilenetV1/Conv2d_3_depthwise/depthwise_bias   INT32     (128)           | 0x00417440 0x00417740 0x00000300
D RKNN: [15:05:44.645] 7   ConvClip const_fold_opt__292                                         INT8      (128,128,1,1)   | 0x0000d300 0x00011300 0x00004000
D RKNN: [15:05:44.645] 7   ConvClip MobilenetV1/MobilenetV1/Conv2d_3_pointwise/Conv2D_bias      INT32     (128)           | 0x00417040 0x00417440 0x00000400
D RKNN: [15:05:44.645] 8   ConvClip const_fold_opt__290                                         INT8      (1,128,3,3)     | 0x00011300 0x00011c00 0x00000900
D RKNN: [15:05:44.645] 8   ConvClip MobilenetV1/MobilenetV1/Conv2d_4_depthwise/depthwise_bias   INT32     (128)           | 0x00416d40 0x00417040 0x00000300
D RKNN: [15:05:44.645] 9   ConvClip const_fold_opt__242                                         INT8      (256,128,1,1)   | 0x00406d40 0x0040ed40 0x00008000
D RKNN: [15:05:44.645] 9   ConvClip MobilenetV1/MobilenetV1/Conv2d_4_pointwise/Conv2D_bias      INT32     (256)           | 0x00416540 0x00416d40 0x00000800
D RKNN: [15:05:44.645] 10  ConvClip const_fold_opt__279                                         INT8      (1,256,3,3)     | 0x0001ae40 0x0001c040 0x00001200
D RKNN: [15:05:44.645] 10  ConvClip MobilenetV1/MobilenetV1/Conv2d_5_depthwise/depthwise_bias   INT32     (256)           | 0x00415f40 0x00416540 0x00000600
D RKNN: [15:05:44.645] 11  ConvClip const_fold_opt__277                                         INT8      (256,256,1,1)   | 0x0001c040 0x0002c040 0x00010000
D RKNN: [15:05:44.645] 11  ConvClip MobilenetV1/MobilenetV1/Conv2d_5_pointwise/Conv2D_bias      INT32     (256)           | 0x00415740 0x00415f40 0x00000800
D RKNN: [15:05:44.645] 12  ConvClip const_fold_opt__301                                         INT8      (1,256,3,3)     | 0x00004c00 0x00005e00 0x00001200
D RKNN: [15:05:44.645] 12  ConvClip MobilenetV1/MobilenetV1/Conv2d_6_depthwise/depthwise_bias   INT32     (256)           | 0x00415140 0x00415740 0x00000600
D RKNN: [15:05:44.645] 13  ConvClip const_fold_opt__273                                         INT8      (512,256,1,1)   | 0x0002c4c0 0x0004c4c0 0x00020000
D RKNN: [15:05:44.645] 13  ConvClip MobilenetV1/MobilenetV1/Conv2d_6_pointwise/Conv2D_bias      INT32     (512)           | 0x00414140 0x00415140 0x00001000
D RKNN: [15:05:44.645] 14  ConvClip const_fold_opt__298                                         INT8      (1,512,3,3)     | 0x00005e00 0x00008200 0x00002400
D RKNN: [15:05:44.645] 14  ConvClip MobilenetV1/MobilenetV1/Conv2d_7_depthwise/depthwise_bias   INT32     (512)           | 0x00413540 0x00414140 0x00000c00
D RKNN: [15:05:44.645] 15  ConvClip const_fold_opt__267                                         INT8      (512,512,1,1)   | 0x0008c4c0 0x000cc4c0 0x00040000
D RKNN: [15:05:44.645] 15  ConvClip MobilenetV1/MobilenetV1/Conv2d_7_pointwise/Conv2D_bias      INT32     (512)           | 0x00412540 0x00413540 0x00001000
D RKNN: [15:05:44.645] 16  ConvClip const_fold_opt__293                                         INT8      (1,512,3,3)     | 0x0000af00 0x0000d300 0x00002400
D RKNN: [15:05:44.645] 16  ConvClip MobilenetV1/MobilenetV1/Conv2d_8_depthwise/depthwise_bias   INT32     (512)           | 0x00411940 0x00412540 0x00000c00
D RKNN: [15:05:44.645] 17  ConvClip const_fold_opt__261                                         INT8      (512,512,1,1)   | 0x000cc4c0 0x0010c4c0 0x00040000
D RKNN: [15:05:44.645] 17  ConvClip MobilenetV1/MobilenetV1/Conv2d_8_pointwise/Conv2D_bias      INT32     (512)           | 0x00410940 0x00411940 0x00001000
D RKNN: [15:05:44.645] 18  ConvClip const_fold_opt__288                                         INT8      (1,512,3,3)     | 0x00011c00 0x00014000 0x00002400
D RKNN: [15:05:44.645] 18  ConvClip MobilenetV1/MobilenetV1/Conv2d_9_depthwise/depthwise_bias   INT32     (512)           | 0x0040fd40 0x00410940 0x00000c00
D RKNN: [15:05:44.645] 19  ConvClip const_fold_opt__258                                         INT8      (512,512,1,1)   | 0x0018c4c0 0x001cc4c0 0x00040000
D RKNN: [15:05:44.645] 19  ConvClip MobilenetV1/MobilenetV1/Conv2d_9_pointwise/Conv2D_bias      INT32     (512)           | 0x0040ed40 0x0040fd40 0x00001000
D RKNN: [15:05:44.645] 20  ConvClip const_fold_opt__305                                         INT8      (1,512,3,3)     | 0x00002000 0x00004400 0x00002400
D RKNN: [15:05:44.645] 20  ConvClip MobilenetV1/MobilenetV1/Conv2d_10_depthwise/depthwise_bias  INT32     (512)           | 0x00420f80 0x00421b80 0x00000c00
D RKNN: [15:05:44.645] 21  ConvClip const_fold_opt__246                                         INT8      (512,512,1,1)   | 0x002c6d40 0x00306d40 0x00040000
D RKNN: [15:05:44.645] 21  ConvClip MobilenetV1/MobilenetV1/Conv2d_10_pointwise/Conv2D_bias     INT32     (512)           | 0x0041ff80 0x00420f80 0x00001000
D RKNN: [15:05:44.645] 22  ConvClip const_fold_opt__283                                         INT8      (1,512,3,3)     | 0x00018800 0x0001ac00 0x00002400
D RKNN: [15:05:44.645] 22  ConvClip MobilenetV1/MobilenetV1/Conv2d_11_depthwise/depthwise_bias  INT32     (512)           | 0x0041f380 0x0041ff80 0x00000c00
D RKNN: [15:05:44.645] 23  ConvClip const_fold_opt__271                                         INT8      (512,512,1,1)   | 0x0004c4c0 0x0008c4c0 0x00040000
D RKNN: [15:05:44.645] 23  ConvClip MobilenetV1/MobilenetV1/Conv2d_11_pointwise/Conv2D_bias     INT32     (512)           | 0x0041e380 0x0041f380 0x00001000
D RKNN: [15:05:44.645] 24  ConvClip const_fold_opt__295                                         INT8      (1,512,3,3)     | 0x00008b00 0x0000af00 0x00002400
D RKNN: [15:05:44.645] 24  ConvClip MobilenetV1/MobilenetV1/Conv2d_12_depthwise/depthwise_bias  INT32     (512)           | 0x0041d780 0x0041e380 0x00000c00
D RKNN: [15:05:44.645] 25  ConvClip const_fold_opt__259                                         INT8      (1024,512,1,1)  | 0x0010c4c0 0x0018c4c0 0x00080000
D RKNN: [15:05:44.645] 25  ConvClip MobilenetV1/MobilenetV1/Conv2d_12_pointwise/Conv2D_bias     INT32     (1024)          | 0x0041b780 0x0041d780 0x00002000
D RKNN: [15:05:44.645] 26  ConvClip const_fold_opt__285                                         INT8      (1,1024,3,3)    | 0x00014000 0x00018800 0x00004800
D RKNN: [15:05:44.645] 26  ConvClip MobilenetV1/MobilenetV1/Conv2d_13_depthwise/depthwise_bias  INT32     (1024)          | 0x00419f80 0x0041b780 0x00001800
D RKNN: [15:05:44.645] 27  ConvClip const_fold_opt__244                                         INT8      (1024,1024,1,1) | 0x00306d40 0x00406d40 0x00100000
D RKNN: [15:05:44.645] 27  ConvClip MobilenetV1/MobilenetV1/Conv2d_13_pointwise/Conv2D_bias     INT32     (1024)          | 0x00417f80 0x00419f80 0x00002000
D RKNN: [15:05:44.645] 28  Conv     MobilenetV1/Logits/AvgPool_1a/AvgPool_2global_2conv_weight0 INT8      (1,1024,7,7)    | 0x00423c40 0x0043c440 0x00018800
D RKNN: [15:05:44.645] 28  Conv     MobilenetV1/Logits/AvgPool_1a/AvgPool_2global_2conv_bias0   INT32     (1024)          | 0x0043c440*0x0043dc40 0x00001800
D RKNN: [15:05:44.645] 29  Conv     const_fold_opt__253                                         INT8      (1001,1024,1,1) | 0x001cc4c0 0x002c68c0 0x000fa400
D RKNN: [15:05:44.645] 29  Conv     MobilenetV1/Logits/Conv2d_1c_1x1/Conv2D_bias                INT32     (1001)          | 0x00421c80 0x00423c00 0x00001f80
D RKNN: [15:05:44.645] 32  Reshape  MobilenetV1/Logits/SpatialSqueeze_2reshape_shape            INT64     (2)             | 0x00423c00 0x00423c40 0x00000040
D RKNN: [15:05:44.645] ---------------------------------------------------------------------------------------------------+---------------------------------
D RKNN: [15:05:44.646] ----------------------------------------
D RKNN: [15:05:44.646] Total Weight Memory Size: 4447296
D RKNN: [15:05:44.646] Total Internal Memory Size: 1756160
D RKNN: [15:05:44.646] Predict Internal Memory RW Amount: 10570545
D RKNN: [15:05:44.646] Predict Weight Memory RW Amount: 4479552
D RKNN: [15:05:44.646] ----------------------------------------
D RKNN: [15:05:44.648] <<<<<<<< end: N4rknn21RKNNMemStatisticsPassE
I rknn buiding done.
done
--> Export rknn model
done
--> Init runtime environment
I target set by user is: rv1106
I Check RV1106 board npu runtime version
I Starting ntp or adb, target is RV1106
I Start adb...
I Connect to Device success!
I NPUTransfer: Starting NPU Transfer Client, Transfer version 2.1.0 (b5861e7@2020-11-23T11:50:36)
D NPUTransfer: Transfer spec = local:transfer_proxy
D NPUTransfer: Transfer interface successfully opened, fd = 3
D RKNNAPI: ==============================================
D RKNNAPI: RKNN VERSION:
D RKNNAPI:   API: 1.5.2 (8babfea build@2023-08-25T02:31:12)
D RKNNAPI:   DRV: rknn_server: 1.5.2 (8babfea build@2023-08-25T10:30:50)
D RKNNAPI:   DRV: rknnrt: 1.2.6b3 (8fc862230@2022-04-28T16:42:02)
D RKNNAPI: ==============================================
D RKNNAPI: Input tensors:
D RKNNAPI:   index=0, name=input, n_dims=4, dims=[1, 224, 224, 3], n_elems=150528, size=150528, w_stride = 0, size_with_stride = 0, fmt=NHWC, type=UINT8, qnt_type=AFFINE, zp=0, scale=0.007812
D RKNNAPI: Output tensors:
D RKNNAPI:   index=0, name=MobilenetV1/Predictions/Reshape_1, n_dims=2, dims=[1, 1001], n_elems=1001, size=2002, w_stride = 0, size_with_stride = 0, fmt=UNDEFINED, type=FP16, qnt_type=NONE, zp=0, scale=1.000000
done
--> Running model
W inference: The 'data_format' has not been set and defaults is nhwc!
mobilenet_v1
-----TOP 5-----
[156]: 0.884765625
[155]: 0.05401611328125
[205]: 0.0036773681640625
[284]: 0.0029735565185546875
[285]: 0.00018918514251708984

done
```

### Other Toolkit Demo
Other Toolkit Demo can be found under `rknn-toolkit2/examples/`, such as quantization, accuracy evaluation, etc. For specific implementation and usage, please refer to the source code and detailed development documents in the Demo.

## Detailed development documentation
CT36L/CT36B For detailed usage of NPU and Toolkit, please refer to the "Rockchip_RKNPU_User_Guide_RKNN_API_\*.pdf" and "Rockchip_User_Guide_RKNN_Toolkit2_\*.pdf" documents under RKNN SDK.

