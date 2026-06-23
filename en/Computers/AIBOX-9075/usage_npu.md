# AI Tutorial

## AidLux Introduction

AidLux is a complete set of edge-end AI development toolkit, which can simplify the environment deployment on the Qualcomm platform and help developers accelerate the implementation of AI applications.

We recommend using AidLux for the development of AI applications. AidLux includes components such as AidLite, AidGen, AidGenSE, and AidStream.

## Install Requirements

* Set AidLux apt source

```bash
# download public key
sudo wget -O- https://archive.aidlux.com/ubuntu24/public.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/private-aidlux.gpg > /dev/null

# edit the source list
sudo vim /etc/apt/sources.list.d/private-aidlux.list

# add this line
deb [arch=arm64 signed-by=/etc/apt/trusted.gpg.d/private-aidlux.gpg] https://archive.aidlux.com/ubuntu24 noble main

# update apt cache
sudo apt update
```

* Install AidLux requirements
```bash
# install these first
sudo apt install python3 python3-pip libopencv-dev python3-opencv  net-tools

# install requirements
sudo apt install aidlux-aistack-base aidrtcm
sudo apt install aid-lms aidlms-sdk cmake aid-mms

# install DSP support
sudo apt-get install qcom-fastrpc1
sudo apt-get install qcom-fastrpc-dev

# install GPU support
sudo apt-add-repository -s ppa:ubuntu-qcom-iot/qcom-ppa
sudo apt install qcom-adreno-cl1
sudo ln -s /usr/lib/aarch64-linux-gnu/libOpenCL.so.1 /usr/lib/aarch64-linux-gnu/libOpenCL.so

# install aidlite-sdk and aidgen-sdk
sudo apt install aidlite-sdk aidlite-*
sudo apt install aidgen-qnn240-sdk
sudo apt-get install libfmt-dev nlohmann-json3-dev
```

After installation, check /usr/local/share/, should have aidlite and aidgen there.

![](../../../qcom_img/AIBOX-9075/check_aid_files.png)

## Model Farm

The Model Farm houses a large number of cutting-edge AI models that have been optimally adapted for use on Qualcomm NPU. Combined with the AidLux toolchain, it can help developers build their own AI applications more quickly on Qualcomm chips.

Addr: https://aiot.aidlux.com/en/models

Please visit Model Farm and register an account, which we will use in the following examples.

## AidLite

AidLite is an AI execution framework designed to fully utilize the computing units (CPU, GPU, NPU) on the edge-side chips to accelerate the inference process of AI models.

The following takes the yolov8s model as an example to demonstrate the usage of AidLite:

* Use mms command to download yolov8s model, need Model Farm account.
```bash
# login, enter the account and password of the Model Farm according to the prompt.
mms login

# list and download model
mms list | grep YOLOv8s | grep FP16 | grep 8550
mms get -m YOLOv8s -p fp16 -c qcs8550 -b qnn2.31 -d ./

# decompress
unzip YOLOv8s_qcs8550_fp16.zip
```

* Navigate to model directory, follow the README to run:
```bash
cd ./code
python3  python/run_test.py --target_model ../models/QCS8550/FP16/cutoff_yolov8s_qcs8550_fp16.qnn231.ctx.bin --imgs ./python/bus.jpg  --invoke_nums 10
```

![](../../../qcom_img/AIBOX-9075/aidlite_run.png)

* Check the result python/result.jpg

![](../../../qcom_img/AIBOX-9075/aidlite_result.png)

## AidGen

AidGen is a dedicated inference framework for generative Transformer models, built based on AidLite. Its purpose is to fully utilize the various computing units (CPU, GPU, NPU) of the hardware to achieve inference acceleration for large models on the edge side.

AidGen offers an atomic-level large model inference interface, which is suitable for developers to integrate large model inference into their own applications.

AidGen supports:

Large Language Model -> AidLLM

Multimodal Large Models -> AidMLM

The following demonstrates the method of deploying LLM with AidGen using the Qwen3-8B-CL8192 model:

* Use mms command to download Qwen3 model, need Model Farm account.
```bash
# login, enter the account and password of the Model Farm according to the prompt.
mms login

# list and download model
mms list | grep Qwen3
mms get -m Qwen3-8B-CL8192 -p W4A16 -c qcs8550 -b qnn2.36 -d ./
```

* Decompress
```bash
mkdir qwen3-8b-aidllm
unzip qnn236_qcs8550_cl8192.zip -d qwen3-8b-aidllm/
```

* Create a model configuration file qwen3-8b-encrypt.json
```bash
cd  ./qwen3-8b-aidllm/qnn236_qcs8550_cl8192
vim qwen3-8b-encrypt.json
```

The contents of configuration file:
```json
{
    "backend_type" : "genie",
    "prefix_path":"kv-cache.primary.qnn-htp",
    "model":{
        "path" : [
        "qwen3-8b_qnn236_qcs8550_cl8192_1_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs8550_cl8192_2_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs8550_cl8192_3_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs8550_cl8192_4_of_5.serialized.bin.aidem",
        "qwen3-8b_qnn236_qcs8550_cl8192_5_of_5.serialized.bin.aidem"
        ]
    }
}
```

* Copy the demo project to model directory then build the demo
```bash
# copy demo project to model directory
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./

# build
cd ./aidllm
mkdir build && cd build
cmake ..
make
```

* Run the demo
```bash
cd ../../
./aidllm/build/test_aidllm  abort ./qwen3-8b-encrypt.json
```

![](../../../qcom_img/AIBOX-9075/aidllm_run.png)

The following demonstrates the method of deploying MLM with AidGen using the Qwen2.5-VL-3B-Instruct (392x392) model

* Use mms command to download Qwen3 model, need Model Farm account.
```bash
# login, enter the account and password of the Model Farm according to the prompt
mms login

# list and download model
mms list | grep Qwen2 | grep VL
mms get -m 'Qwen2.5-VL-3B-Instruct (392x392)' -p w4a16 -c qcs8550 -b qnn2.36 -d ./
```

* Decompress
```bash
mkdir qwen2.5-7b-aidmlm
unzip qnn236_qcs8550_cl2048.zip -d ./qwen2.5-vl-aidmlm/
```

* Create a model configuration file Qwen2.5-VL-3B-392x392-8550.json
```bash
cd ./qwen2.5-vl-aidmlm/qnn236_qcs8550_cl2048/
vim Qwen2.5-VL-3B-392x392-8550.json
```

The contents of configuration file:
```json
{
    "vision_model_path": "./veg.serialized.bin.aidem",
    "pos_embed_cos_path": "./position_ids_cos.raw",
    "pos_embed_sin_path": "./position_ids_sin.raw",
    "vocab_embed_path": "./embedding_weights_151936x2048.raw",
    "window_attention_mask_path": "./window_attention_mask.raw",
    "full_attention_mask_path": "./full_attention_mask.raw",
    "llm_path_list": [
        "./qwen2p5-vl-3b_qnn236_qcs8550_cl2048_1_of_1.serialized.bin.aidem"
    ]
}
```

* Copy demo project to model directory then build the demo
```bash
# copy demo project to model directory
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidmlm/* ./

# build
mkdir build && cd build
cmake ..
make
```

* Run the demo
```bash
cd ..
./build/test_aidmlm single qwen25vl3b392 Qwen2.5-VL-3B-392x392-8550.json /home/ubuntu/dog.jpg "Please describe this picture."
```

![](../../../qcom_img/AIBOX-9075/aidmlm_run.png)

## AidGenSE

AidGenSE is a generative AI HTTP service that is based on AidGen and has been adapted to the OpenAI HTTP protocol. Developers can call the generative AI through HTTP and quickly integrate it into their own applications.

The following is an introduction to the usage of AidGenSE:

* Prepare AidGenSE environment
```bash
# setup python virtual env
sudo apt install -y python3-pip python3-venv > /dev/null 2>&1
sudo python3 -m venv /opt/aidlux/aid-python3

# create aid-python3 command
echo '#!/bin/bash
exec /opt/aidlux/aid-python3/bin/python3 "$@"' | sudo tee /usr/bin/aid-python3 > /dev/null
sudo chmod +x /usr/bin/aid-python3

# create aid-pip3 command
echo '#!/bin/bash
exec /opt/aidlux/aid-python3/bin/python3 -m pip "$@"' | sudo tee /usr/bin/aid-pip3 > /dev/null
sudo chmod +x /usr/bin/aid-pip3

# install AidGenSE
sudo apt install aidgense
sudo aidllm system --sys linux --soc 8550
sudo apt install aid-pkg
```

* Common instructions
```bash
# list available models
sudo aidllm remote-list api
# download model
sudo aidllm pull api aplux/Qwen3-8B-8550-cl2048
# list downloaded models
sudo aidllm list api 
# start model api
sudo aidllm start api
# stop model api
sudo aidllm stop api 
```

* After "start api", you can use OpenAI HTTP requests to interact with AI model through <device ip>:8888

![](../../../qcom_img/AIBOX-9075/aidllm_start_api.png)

## AidStream

AidStream is a multimedia data real-time processing toolkit based on GStreamer, suitable for building AI-based video and image analysis applications and services.

Demo:

* Install requirements
```bash
sudo apt-add-repository -s ppa:ubuntu-qcom-iot/qcom-ppa
sudo apt update
sudo apt install weston-autostart

sudo apt install gstreamer1.0-qcom-sample-apps
sudo reboot # need reboot once

sudo apt-get install libgstreamer1.0-dev gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-alsa gstreamer1.0-gtk3

sudo apt-get install -y libjsoncpp-dev

sudo ln -s /usr/lib/aarch64-linux-gnu/libjsoncpp.so /usr/lib/aarch64-linux-gnu/libjsoncpp.so.1

sudo apt install gstreamer1.0-rtsp

sudo apt install aidstream-gst
```

* Configure video stream input and output addresses.
```bash
cd /usr/local/share/aidstream-gst/conf
# modify the input and output addresses to available video stream server addresses.
vim aidstream-gst.conf
```

![](../../../qcom_img/AIBOX-9075/aidstream_conf.png)

* Run the demo
```bash
mkdir aid-streamer-gst
cd aid-streamer-gst
cp -r /usr/local/share/aidstream-gst/example/ ./
cd example/python
sudo python3 example.py
```

Use the output address to check the result.