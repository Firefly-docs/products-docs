# ChatGLM2

AIBOX-1684 可部署 int4 或 int8 量化的 ChatGLM2 大语言模型。

## 部署

（1）从[下载中心]获取部署 ChatGLM2 所需的部署压缩包。注：若想尝试自行编译模型和程序，可按照本文后续的编译章节进行操作。

（2）将压缩包发送到机器上并解压。

（3）安装部署所需的 Python 依赖库：

```shell
pip3 install gradio=3.40.0
pip3 install mdtex2html
```

（4）运行 `python3 web_demo.py`，等待至终端显示如下字样时，服务启动成功：

```shell
Running on local URL:  http://0.0.0.0:7860

To create a public link, set `share=True` in `launch()`.
```

（5）使用浏览器访问 http://1684x_address:7860/，其中 1684x_address 改为 1684x 机器的ip地址，成功进入页面后即可使用 ChatGLM2。

## 编译模型

（1）配置 Docker 环境：

注：模型编译操作均在该 docker 容器中进行

```shell
sudo docker pull sophgo/tpuc_dev:v3.1
sudo docker run --name chatglm2 -v $PWD:/workspace -it sophgo/tpuc_dev:v3.1

apt update
apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

（2）拉取 ChatGLM2-6B 模型仓库并修改：

```shell
# 安装 git 的 lfs 服务
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
apt install git-lfs
git lfs install

# 从 hugging face 拉取模型仓库，需配置 ssh 公钥，https://huggingface.co/THUDM/chatglm2-6b
git clone git@hf.co:THUDM/chatglm2-6b
```

修改仓库：

* 将 config.json 文件中的 seq_length 属性设置为512。

* 将 modeling_chatglm.py 文件中的如下代码：

```shell
if attention_mask is not None:
    attention_scores = attention_scores.masked_fill(attention_mask, float("-inf"))
```

改为：

```shell
if attention_mask is not None:
    attention_scores = attention_scores + (attention_mask * -10000.0)
```

* 将 modeling_chatglm.py 文件中的如下代码：

```shell
pytorch_major_version = int(torch.__version__.split('.')[0])
if pytorch_major_version >= 2:
```

改为：

```shell
pytorch_major_version = int(torch.__version__.split('.')[0])
if False:
```

（3）配置 TPU-MLIR 编译器：

```shell
git clone git@github.com:sophgo/tpu-mlir.git
cd tpu-mlir
source ./envsetup.sh
./build.sh

export PYTHONPATH=/workspace/chatglm2-6b:$PYTHONPATH
```

（4）配置 ChatGLM2-TPU 编译工程仓库：

```shell
git clone git@github.com:sophgo/ChatGLM2-TPU.git

pip3 install sentencepiece
```

（5） 编译

```shell
cd ChatGLM2-TPU/compile
python3 export_onnx.py

# 编译 int4 或 int8 量化模型
./compile.sh --mode int4 # or int8
```

## 编译部署程序

注：部署程序的编译过程均在版本为20.04的 Ubuntu 系统中进行。

（1）安装交叉编译工具 `sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu`。

（2）准备 libsophon 依赖库:

```shell
# 在 1684x 机器中进行操作
cd /opt/sophon/

# 将当前目录下的 libsophon-x.x.x 文件夹发送至编译环境的工作目录，示例：
# scp -r libsophon-0.4.8/ username@ip_address:~/chatglm2/libsophon
```

（3）编译分词器依赖库：

```shell
git clone git@github.com:google/sentencepiece.git
cd sentencepiece

# 在 CMakeLists.txt 中添加以下代码，以实现交叉编译
set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_ASM_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)
# -----------------------------------------------------------

mkdir build
cd build
cmake ..
make -j
# 需要的文件位于 ~/sentencepiece/build/src/libsentencepiece.a
```

（4）更换 ChatGLM2-TPU 仓库中的分词器依赖库，用 `~/sentencepiece/build/src/libsentencepiece.a` 替换掉 `~/ChatGLM2-TPU/web_demo/lib/libsentencepiece.a`。

（5）修改 CMakeLists.txt。

添加以下代码：

```shell
set(CMAKE_C_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_ASM_COMPILER aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)
```

将  CMakeLists.txt 中的：

```shell
set(LIBSOPHON /opt/sophon/libsophon-current)
```

改为：

```shell
set(LIBSOPHON ~/libsophon)
```

（6）编译

```shell
mkdir build
cd build
cmake ..
make -j
```

[下载中心]: https://www.t-firefly.com/doc/download/280.html