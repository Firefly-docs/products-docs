
## OpenCV 编译和安装

不少客户遇到OpenCV的问题多集中在如何获取mipi摄像头的数据。因为OpenCV使用的V4l2协议和Rockchip编写的mipi摄像头驱动协议不同，所以不能直接使用OpenCV的摄像头API。本节讲述如何编译Opencv并添加GStreamer API支持。

### 要求
* OS： Ubuntu18.04 / Debian 10
* OpenCV version： 3.4.15
* Board: RK3399

### 编译和安装
1. 搭建python3.7虚拟环境
    ```
    # 依次安装gcc和cmake等编译环境
    # 安装python3.7-tk 和 python3.7-dev
    # 安装virtualenv虚拟环境
    sudo apt install gcc cmake git build-essential \
    python3-tk python3.7-dev \
    virtualenv
    ```

1. 创建python3.7虚拟环境
    ```
    virtualenv -p /usr/bin/python3.7m /home/firefly/venv
    # 使用虚拟环境，如果想退出可以在终端输入 deactivate
    source /home/firefly/venv/bin/activate
    ```

1. 安装Opencv所需的环境包
    ```
    # 安装编译环境、gtk包和相关编解码库
    sudo apt install cmake build-essential libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev

    # 注意！如果安装了rknn_toolkit(or lite)则不需要再安装Python-numpy，否则会导致rknn_toolkit(or lite)无法使用
    sudo apt install python-numpy
    ```

1. 下载Opencv
    ```
    # 创建文件夹
    mkdir opencv
    cd opencv

    # 下载 opencv-3.4.15.zip
    wget https://github.com/opencv/opencv/archive/refs/tags/3.4.15.zip
    unzip opencv-3.4.15.zip
    ```

1. 配置
    ```
    mkdir build
    cd build

    # 让 OpenCV 和 Python3 之间建立联系
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D 	PYTHON_DEFAULT_EXECUTABLE=$(python -c "import sys; print(sys.executable)")\
    -D PYTHON3_EXECUTABLE=$(python -c "import sys; print(sys.executable)")  \
    -D PYTHON3_NUMPY_INCLUDE_DIRS=$(python -c "import numpy; print (numpy.get_include())")  \
    -D PYTHON3_PACKAGES_PATH=$(python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())")  \
    ../opencv-3.4.15
    ```

1. 如需支持Gstreamer API，则进行以下配置操作否则跳过
    ```
    sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
    cmake -D WITH_GSTREAMER=ON ../opencv-3.4.15
    ```
1. 编译安装
    ```
    # 如果板子内存不超过2G，建议不超过-j4
    make -j6
    # 安装的过程需要sudo权限
    sudo make install
    ```

### 测试
1. 测试
    ```
    source /home/firefly/venv/bin/activate
    git clone https://gitlab.com/firefly-linux/test_code/rkisp-v4l2.git
    cd mipi_video_demo/OpenCV_Python
    python3 opencv_gst_test.py
    ```

2. opencv_gst_test.py 代码
    ```
    import numpy as np
    import cv2 as cv
    import os
    import time

    cap = cv.VideoCapture('v4l2src device=/dev/video1 ! video/x-raw, format=NV12, width=640, height=480, framerate=30/1 ! videoconvert ! appsink', cv.CAP_GSTREAMER)

    if not cap.isOpened():
        print("Cannot capture from camera. Exiting.")
        os._exit()
    last_time = time.time()

    while(True):

        ret, frame = cap.read()
        this_time = time.time()
        print (str((this_time-last_time)*1000)+'ms')
        last_time = this_time;
        cv.imshow('frame', frame)

        if cv.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv.destroyAllWindows()
    ```