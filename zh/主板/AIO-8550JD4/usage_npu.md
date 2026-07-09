# AI 教程

QCS8550 拥有 Hexagon 数据处理芯片(DSP)，里面包含用于处理 AI 计算的 NPU。

所以一般情况下 NPU 或 DSP 都是指这同一颗芯片，并不是两个不同的芯片。

## Aidlux 介绍

Aidlux 是一套完备的边缘端 AI 开发工具套件，能简化高通平台的环境部署，帮助开发者加速 AI 应用落地。

目前我们使用 docker 的方式部署 Aidlux Web Desktop，来进行开发。

AIBOX 出厂一般会预装 Aidlux Web Desktop，通过如下命令查看：
```bash
docker ps -a

CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS     NAMES
xxxxxxxxxxxx   aidlux_lu_ubun2204:xxx   "/lib/systemd/systemd"   5 minutes ago   Up 5 minutes             aidlux
```

如果存在名为 aidlux 的容器，则说明已经预装，可以直接使用。

如果不存在则需要进行下载安装：

## Aidlux Web Desktop 安装

前往 [下载地址](https://www.t-firefly.com/doc/download/356.html#other_931) 下载，将压缩包传到设备上，解压后执行里面的 setup.sh 即可。
```bash
# 解压
tar -zxf aidlux_docker.tgz

# 安装
cd aidlux_docker
chmod + setup.sh
./setup.sh
```

安装过程需要约 5 分钟，安装后容器会自动启动。并且之后也会开机自动启动，无需再操作。

## Aidlux Web Desktop 使用

将设备 eth0 联网，执行 ifconfig eth0 查看设备 ip 地址。

在局域网内任意电脑打开浏览器，输入 `设备ip:8000` 并访问。比如 `172.16.10.100:8000`，就可看到 Web Desktop 的画面。

![](../../../qcom_img/aidlux_login.jpg)

登陆密码是 aidlux

Aidlux Web Desktop 的操作方法和 Linux 桌面差别不大，可以使用终端、应用商店等等。

下面的操作均在 Web Desktop 中完成：

## AI 开发

### Aidlite

Aidlite 是 AI 执行框架，可以调用高通平台的 CPU/GPU/NPU(DSP) 进行模型加速推理。

打开终端，前往 `/usr/local/share/aidlite/examples` 可以看到一些例子：

![](../../../qcom_img/aidlite_demos.jpg)

比如我们可以尝试运行 aidlite_qnn236 的 python 例子
```bash
cd aidlite_qnn236/python/

# 运行，其中 3 表示使用 NPU(DSP) 进行加速。sudo 密码是 aidlux
sudo python qnn_yolov5_multi.py 3
```

运行完成会在当前目录生成结果图片 qnn_yolov5_multi.jpg

可以使用文件浏览器查看生成的图片结果：

![](../../../qcom_img/aidlite_demo_result_location.jpg)

![](../../../qcom_img/aidlite_demo_result_img.jpg)

### AidGenSE

AidGenSE 是适配了 OpenAI HTTP 协议的生成式 AI HTTP 服务。开发者可以通过 HTTP 方式调用生成式 AI 并快速集成到自己的应用中。

首先查看是否有预装大模型：
```bash
# 列出已下载的模型
aidllm list api

qwen2.5-3B-Instruct-8550
```

如果返回空或者报错命令不存在，则需要进行安装：

* 安装

安装 AidGenSE
```bash
sudo aid-pkg update
sudo aid-pkg -i aidgense
```

安装 UI
```bash
sudo aidllm install ui
```

查看支持的模型：
```bash
aidllm remote-list api
```

根据模型的 Url 获取模型，这里以 `aplux/qwen2.5-3B-Instruct-8550` 为例：
```bash
aidllm pull api aplux/qwen2.5-3B-Instruct-8550
```

* 对话示例

模型下载完成后，启动服务：
```
# 启动对应模型的 openai api 服务
# aidllm start api -m <model_name> 
aidllm start api -m qwen2.5-3B-Instruct-8550

# 其他命令如下
# 查看状态
aidllm status api

# 停止服务
aidllm stop api

# 重启服务
aidllm restart api
```

启动服务后，打开桌面上的 NextChat 即可开始对话

![](../../../qcom_img/aidlux_nextchat.jpg)

## 其他

### 文件传输

可以使用文件浏览器从电脑中向 Web Desktop 中上传文件，也可以下载文件到电脑。

打开文件浏览器，文件浏览器会默认打开 `/home/aidlux` 文件夹。只有这个文件夹可以接收上传的文件。点击右上角的上传按钮即可上传文件或文件夹。

![](../../../qcom_img/aidlux_upload_file.jpg)

选中文件，点击右上角下载按钮，即可下载文件。

![](../../../qcom_img/aidlux_download_file.jpg)

### 更多教程

有关 AI 开发的更多教程，可以前往 [Aidlux 文档中心](https://rhinopi.docs.aidlux.com/software/) 查看。