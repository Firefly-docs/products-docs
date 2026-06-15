# 技术案例

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
## 容器虚拟安卓

AIC (Android in Container) 是指在 Linux 系统中使用容器来运行安卓。RK3576 Linux 即可使用 docker 来运行安卓并支持多开。

集群服务器搭配 AIC 可以增加安卓的密度，在云手机、云游戏等场景有很大的帮助。

此处有固件可供测试：[点击前往](https://pan.baidu.com/s/1sbZwOc4peZfn0HDX9O0lsg) 提取码 1234

### 使用方法

说明：

1. 固件使用的宿主系统是 Ubuntu Minimal 版本，使用 debug 串口、ssh、adb shell 等方式进行交互，没有桌面显示。
2. 安卓的交互是通过网络 adb 连接并配合 scrcpy 等投屏工具实现，不是接入鼠标键盘来操作。
3. 操作 AIC 需要知道基础的 Linux 终端操作知识、docker 容器知识、adb 使用方法。

#### 创建容器

固件自带 docker 镜像，只需要创建容器。/root/ 目录下存在一系列脚本 docker_sh，执行脚本进行创建：
```bash
./docker_sh/run_android.sh rk3588:firefly <id> <ipv4_address>

# 例如
./docker_sh/run_android.sh rk3588:firefly 0 192.168.100.1
```
id 是人为给容器设置的编号，它会以后缀的形式加在容器名称末尾，方便管理。

ipv4_address 是给容器指定的 ip 地址，**它需要与宿主机处于同网段并且不和其他设备或容器冲突**。

需要多开安卓就多执行几遍，注意编号和 ip 不能相同。

#### 连接容器

在局域网中使用任意电脑通过 adb 即可访问容器。
```bash
# adb 连接容器，这里的 ip 就是容器 ip
adb connect <ip>

# 在上一步连接成功后
# 投屏显示
scrcpy -s <ip>
```
adb 和 scrcpy 的安装方法请自行上网搜索。

#### 管理容器

使用 docker 命令管理容器即可。
```bash
# 查看所有容器
root@firefly:~# docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED      STATUS                    PORTS     NAMES
cad5a331dea9   rk3588:firefly   "/init androidboot.h…"   6 days ago   Exited (137) 6 days ago             android_1
37f60c3b6b80   rk3588:firefly   "/init androidboot.h…"   6 days ago   Up 13 seconds                       android_0

# 启动/停止容器
docker start/stop <NAMES>

# 连接到安卓终端(再次执行 exit 退出)
docker exec -it <NAMES> sh

# 删除容器
docker rm <NAMES>
```
如果宿主机网段发生变化，那么需要重新设置 macvlan 并修改容器 ip 地址。

根据实际情况修改参数，可参考 ./docker_sh/run_android.sh
```bash
docker network rm macvlan
docker network create -d macvlan --subnet=<SUBNET> --gateway=<GATEWAY> -o macvlan_mode=bridge -o parent=<PARENT> macvlan
```
修改容器 ip 的方法请上网搜索，这里不介绍。

### 性能展示

以 root 身份执行以下命令开启性能模式可以获得更好的体验：
```bash
# 出现一个 Invalid argument 是正常的，不用管
root@firefly:~# echo performance | tee $(find /sys/devices -name *governor)
performance
tee: /sys/devices/system/cpu/cpuidle/current_governor: Invalid argument
```

进入安卓的终端或使用 adb shell 执行如下命令可以打印游戏渲染帧率：
```bash
# 注意，只有在游戏运行中，这个命令才会打印帧率
setprop debug.sf.fps 1;logcat -s SurfaceFlinger
```

单个 RK3576 使用 AIC 同时运行两个最高画质原神可达 35 帧以上:
![](../../../rk3576_img/common/aic_ys_performance.png)
## Rockit AVS

<font color=red>**平台限制：目前只在 3588 平台 Buildroot 系统下验证支持。如需使用请把 external/rockit 仓库切换到远程的 rk3588/avs 分支。**</font>

### AVS 在 Buildroot 系统适配

源码路径在`sdk/external/rockit/mpi/example/mod`，参考 Rockit AVS 添加了 firefly_mpi_avs_test 程序样例，采用6路输入等距柱面投影非压缩融合拼接模式，添加拼接图片素材、标定文件及程序运行文件输入输出路径，已整合到 buildroot 中。

```bash
diff --git a/mpi/example/mod/CMakeLists.txt b/mpi/example/mod/CMakeLists.txt
index 679c0b0..b6891df 100644
--- a/mpi/example/mod/CMakeLists.txt
+++ b/mpi/example/mod/CMakeLists.txt
@@ -76,6 +76,10 @@ set(RK_MPI_TEST_AVIO_SRC
     sys/test_sys_avio.cpp
 )

+set(FIREFLY_MPI_TEST_AVS_SRC
+    firefly_test_mpi_avs.cpp
+)
+
 #--------------------------
 # rk_mpi_ao_test
 #--------------------------
@@ -199,3 +203,15 @@ install(TARGETS rk_mpi_gdc_test RUNTIME DESTINATION "bin")
 add_executable(rk_mpi_avio_test ${RK_MPI_TEST_AVIO_SRC} ${RK_MPI_TEST_COMMON_SRC})
 target_link_libraries(rk_mpi_avio_test ${ROCKIT_DEP_COMMON_LIBS})
 install(TARGETS rk_mpi_avio_test RUNTIME DESTINATION "bin")
+
+#--------------------------
+# firefly_mpi_avs_test
+#--------------------------
+add_executable(firefly_mpi_avs_test ${FIREFLY_MPI_TEST_AVS_SRC} ${RK_MPI_TEST_COMMON_SRC})
+target_link_libraries(firefly_mpi_avs_test ${ROCKIT_DEP_COMMON_LIBS})
+install(TARGETS firefly_mpi_avs_test RUNTIME DESTINATION "bin")
+
+#--------------------------
+# add 6x_rectlinear data
+#--------------------------
+install(DIRECTORY "6x_rectlinear" DESTINATION "data/avs")
diff --git a/mpi/example/mod/firefly_test_mpi_avs.cpp b/mpi/example/mod/firefly_test_mpi_avs.cpp
new file mode 100644
index 0000000..9cfb68c
--- /dev/null
+++ b/mpi/example/mod/firefly_test_mpi_avs.cpp
```

### AVS 使用说明

1.文件路径说明

```bash
#拼接素材路径
/usr/data/avs/6x_rectlinear/input_image/image_data/
#输出路径
/usr/data/avs/6x_rectlinear/output_res/
#标定文件
/usr/data/avs/6x_rectlinear/avs_calib/calib_file.pto
```

2.运行说明

执行如下命令：

```bash
root@RK3588:/# chmod 777 /usr/data/avs/6x_rectlinear/firefly_prepare_avs_env.sh
root@RK3588:/# /usr/data/avs/6x_rectlinear/firefly_prepare_avs_env.sh
root@RK3588:/# firefly_mpi_avs_test
```

程序运行时重复输出拼接图片的 bin 格式文件到指定路径，默认执行 100 次。

```bash
output: /usr/data/avs/6x_rectlinear/output_res/chn_out_8192x2700_0_0_nv12.bin
```

若程序出现库缺失或者异常卡死情况，请将指定库文件拷贝到`/usr/lib/`下：

```bash
#将指定库文件拷贝到/usr/lib/下
cp /usr/data/avs/6x_rectlinear/lib/libgraphic_lsf /usr/lib/libgraphic_lsf.so
cp /usr/data/avs/6x_rectlinear/lib/libpanoStitchApp /usr/lib/libpanoStitchApp.so
```

使用 YUView 工具（Windows 工具）可查看素材路径下的 YUV 图片，demo 运行后在输出路径下生成 bin 格式文件，将 bin 文件拖进 YUView-Item 栏，设置 W、H、YUVformat 等选项查看 YUV 图片（8192x2700-YUV(IL) 4:2:0 8-bit），预览情况如下：

![](../../../rk3576_img/avs_yuviewtool_preview.png)

3.素材制作

demo 采用图片输入格式为 yuv-nv12，可将 jpg 图片素材转为 yuv-nv12 格式进行测试，需要自行标定，标定文件为`calib_file.pto`，包含了图片大小、偏航角、俯仰角、翻滚角等拼接参数信息。

```bash
#jpg转yuv-nv12
ffmpeg -i camera_0.jpg -s 1520x2560 -pix_fmt nv12 camera_0.yuv
```

标定文件如下：
```bash
root@RK3588:/usr/data/avs/6x_rectlinear/avs_calib# cat calib_file.pto

p w8378 h4189 f2 v360 u0 n"JPEG g0 q95"
m g0 i0

i n"camera_0.jpg" w2560 h1520 f0 y-113.603 r-90.9529 p30.5159 v90.12 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_1.jpg" w2560 h1520 f0 y-68.4270 r89.24700 p32.1190 v90.24 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_2.jpg" w2560 h1520 f0 y-26.2000 r-90.0960 p32.8350 v90.16 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_3.jpg" w2560 h1520 f0 y22.40400 r90.50000 p32.8000 v90.29 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_4.jpg" w2560 h1520 f0 y63.53500 r-89.1020 p31.5140 v90.26 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_5.jpg" w2560 h1520 f0 y110.5400 r93.83400 p31.6180 v90.53 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0
```

### 拼接性能测试

GPU 定频到 1G，在程序运行时使用 dumpsys avs 命令统计图片数据的吞吐量以分析拼接性能。

```bash
#设置userspace策略定频gpu，然后设置不同频率，比较设置电压和实测电压
echo userspace > /sys/class/devfreq/ff400000.gpu/governor
#设置定频的频率
echo 1000000000 > /sys/class/devfreq/ff400000.gpu/userspace/set_freq
```

拼接性能方面，输出分辨率可接近 22M@30fps（8192x2700），测试 log 如下：

```bash
-------------------------------------------------------------------------------
END DUMP OF SERVICE avs:
# dumpsys avs
-------------------------------------------------------------------------------
DUMP OF SERVICE avs:
---------------------- avs group attr ----------------------------
grp_id    mode           enable    pipe_num  is_sync   src_rate  dst_rate
0         BLEND          Y         6         N         -1        -1
---------------------- avs lut attr ----------------------------
grp_id    lut_data_acc   lut_data_path
0         HIGH           NONE
---------------------- avs output attr ----------------------------
grp_id    proj_mode                center_x  center_y  fov_x     fov_y
0         EQUIRECTANGULAR          4220      2124      28000     9500

          ori_yaw   ori_pitch ori_roll  yaw       pitch     roll
0         0         0         0         0         0         0

          middel_lut_path     calib_path          mask_path           mesh_alpha_path
0         NONE                /data/avs/6x_rectlinNONE                /data/avs/6x_rectlin
                              ear/avs_calib/calib_                    ear/avs_mesh/
                              file.pto
---------------------- avs channel attr ----------------------------
grp_id    chn_id    enable    width     height    is_compress    dym_range depth     src_rate  dst_rate  
0         0         Y         8192      2700      N              SDR8      3         -1        -1
---------------------- avs group work status ----------------------------
grp_id    cost_time max_cost_time  false_count
0         34        40             0
---------------------- avs pipe work status ----------------------------
grp_id    pipe_id_0 pipe_id_1 pipe_id_2 pipe_id_3 pipe_id_4 pipe_id_5 pipe_id_6 pipe_id_7
0         100       100       100       100       100       100       0         0
          0         0         0         0         0         0         0         0
---------------------- avs pipe queue ----------------------------
grp_id    pipe_id_0 pipe_id_1 pipe_id_2 pipe_id_3 pipe_id_4 pipe_id_5 pipe_id_6 pipe_id_7
```

- 相关参数说明：
  - pipe_num：Pipe 数目
  - proj_mode：EQUIRECTANGULAR
  - cost_time：当前拼接任务耗时(ms)，拼接输出帧率 fps = 1000(ms) / cost_time(ms)，当 cost_time 接近 33ms 时，fps 为 30fps
  - max_cost_time：拼接历史最大耗时(ms)
  - pipe_id_0-pipe_id_7：对应输入 Pipe 上的接收的图像数/对应输入 Pipe 上的丢弃的图像数