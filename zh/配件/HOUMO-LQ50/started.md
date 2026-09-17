# 后摩 LQ50 AI 计算卡

## 产品介绍

### 产品简介



| 正面 | 反面 |
| :---: | :---: |
| ![](../../../modules_img/HOUMO-LQ50/houmo-lq50-1.png) | ![](../../../modules_img/HOUMO-LQ50/houmo-lq50-2.png) |

### 详细参数

请联系销售 (sales@t-firefly.com) 获取对应模块规格书。



## 使用方法

考虑到后摩模块资源主要以 docker 容器进行测试，体积较大。同时驱动安装为DKMS自动编译安装，需要板端第一次安装时有网络并配齐对应开发环境。因此 Firefly 发布两种形式的资源。一种是 Firefly 基于自身固件，将编译好的驱动和测试工具直接打包好，让使用 Firefly 固件的客户能直接测试（不进行长期维护，主要用于检测模块是否正常）。另一种则为 Houmo 官方资源包，完成初步对 Houmo模块的测试后，客户可以再根据需求进行下载对应的资源。两种资源的驱动安装方式不同：Firefly 安装包内的驱动已编译好，直接运行安装脚本即可；官方资源包则需先配置编译环境，再运行 .run 安装。官方资源包装好驱动后，后续使用又分为两条路径：**依赖 Docker 镜像**与**裸板（不依赖 Docker）**，见[安装驱动](#安装驱动)小节



### Firefly Houmo 安装包

内部包括

1. 编译好的Houmo LQ50 模块 aarch64 平台驱动（考虑到大小，裁剪了GUI工具）
2. 测试包
3. runtime 压缩包
4. 安装脚本

运行install.sh，会自动安装驱动，并将 runtime 以 pip 方式安装到系统 Python（源码包现场编译 tcim_lite，需要网络；缺失的编译依赖脚本会自动补装）。解压测试包，运行run_all.sh，即可进行测试，测试默认只运行 check 测试，用于检测模块与环境是否正常。可运行run_all.sh --aging 进行 2 hour 压力测试。会分别测试模块的DDR读写速率，算力， 以及推理性能（推理性能验证使用的是CNN模型，主要用于验证模块的功能完整）。在 venv 中做模型开发时，需在 venv 内再 pip 安装一次包内的 runtime 压缩包（见[路径二：裸板](#路径二裸板不依赖-docker)的「安装 runtime」小节）；houmo_test 二进制测试工具会自动定位 pip 安装的库，无需 /opt。

安装后，运行`hm_smi -a`测试，若能检测到模块，则请直接跳转到[路径一：依赖 Docker 镜像](#路径一依赖-docker-镜像)的「模块测试」小节。无需再次配置驱动与运行时环境。

> 注意，Firefly Houmo 安装包仅支持 Firefly 所提供支持的固件，主要是内核版本的原因，因此在使用时请注意。



### 官方资源包下载

请联系销售 (sales@t-firefly.com) 获取 **Houmo 网盘资料** 下载链接，根据需求，选择相应 Houmo 版本进行下载。网盘中有上传的后摩资源，在我们所提供的固件上均已经过验证（如 Debian 12).

**下载资源**

```bash
houmo-drv-<target_hw>_<release>_${distro}_$arch.run			# 驱动包
houmo_tcim_runtime_xh2_linux_aarch64-<version>.tar.gz		# Python runtime 源码包（裸板 pip 安装用）
Dadao-deploy-docker-xh2-vx.y.z-<rootfs>-aarch64.tar			# 测试环境镜像
houmo-examples-xh2_<version>.zip		# 测试程序包
```

详细的包命名介绍，请参考[3.1. Linux主机端安装与部署 — M50 软件平台快速入门 1.4.0 文档](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/quickstart/getting-started/quickstart_guide/setup/linux.html) 与 [3.2. Ubuntu/Kylin V11/UOS驱动安装与卸载 — M50 软件平台驱动安装指南 1.4.0 文档](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/system-installation-device-management/environment-deployment/system_software_installation_guide/linux/ubuntu.html)





### 安装驱动

驱动安装分两种方式：**Firefly 安装包**内的驱动已编译好，直接运行安装脚本即可；**官方资源包**的 .run 安装脚本通过 DKMS 在板端现场编译，需先配置编译环境并有网络。

#### 方式一：Firefly 安装包（免配环境，直接运行安装）

安装包获取与包内组成见 [Firefly Houmo 安装包](#firefly-houmo-安装包) 小节。在解压后的包根目录运行 install.sh，即可完成驱动安装，并将 runtime 安装到系统 Python（runtime 为源码包现场编译，需要网络）：

```bash
tar -xzf firefly_houmo_v*.tar.gz
cd firefly_houmo_v*/
sudo ./install.sh
```

若只需要驱动，也可以直接安装包内的 deb（零编译、不访问网络，安装后自动完成 depmod、modprobe、udev 重载、ldconfig，并进行 hm_smi 自检）：

```bash
sudo dpkg -i houmo-installer-xh2-v*.deb
```

#### 方式二：官方资源包（需配环境并运行 .run）

如下教程，都以V1.2.0版本举例，请根据需求替换。

```bash
# 安装加速卡 pcie 驱动
apt  update
sudo apt-get install python3 python3-dev python3-pip -y
apt install /boot/linux-headers-6.1-arm64_arm64.deb
# pip 换国内源（华为云/阿里实测较快；清华源在部分网络下 403 不可用）
pip3 config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple
pip3 config set global.extra-index-url https://mirrors.aliyun.com/pypi/simple/
chmod a+x houmo-drv-xh2_v1.2.0_linux_aarch64.run 
sudo bash ./houmo-drv-xh2_v1.2.0_linux_aarch64.run --build-host false install all
```

> **报错，找不到linux-headers-6.1**
>
> ```
> cd /boot
> ls
> ```
>
> 查看板子对应的版本
>
> 执行如下指令
>
> ```bash
> dpkg -i /boot/linux-headers-<version>-arm64_arm64.deb # 替换为对应版本的deb名称
> ```
>
> 重新运行
>
> ```bash
> sudo bash ./houmo-drv-xh2_v1.2.0_linux_aarch64.run --build-host false install all
> ```

安装完成后，运行

```bash
hm_smi -a
```

应该能够看到类似如下形式的打印

```
--------------------------------------------------------------------------------
  sdk build infos
--------------------------------------------------------------------------------
  Build_Time     : 2026-08-20 19:40:00
  HMSW_Version   : V1.2.0
  HM_SMI_Version : V1.0.0
--------------------------------------------------------------------------------
  Wed Sep 16 05:35:51 CST 2026
--------------------------------------------------------------------------------
  device0 detail infos
--------------------------------------------------------------------------------
  Driver_Version         : V1.2.0
  Vendor                 : Houmo
  BDF                    : 0004:41:00.0
  Dev                    : 0
  Cur_BandWidth          : 5.0 GT/s x 1lane
  Power_Management       :
    DVFS_Mode            : performance
    Cur_Ipu_Freq         : 1400.0 Mhz
    Lock_Ipu_Max_Clock   : 1400.0 Mhz
    Lock_Ipu_Min_Clock   : 700.0 Mhz
    IPU_Load             : 0.0 %
  Firmware_Version       : V1.1.1
  IPU_Infos              :
    Core_Num             : 2
    Core_Freq            : 1400.0 Mhz
    Voltage              : 750.0 mV
    Core0_Util           : 0.0 %
    Core1_Util           : 0.0 %
    Average_Util         : 0.0 %
  Group_Id               : 0
  Chip_Id                : 0
  SN                     : 0104020100002026001000000522
  PN                     : 100O2010
  Model                  : LQ50-24GB
  DDR_Memory_Infos       :
    DDR_Memory_Free      : 24448.0MB
    DDR_Memory_Total     : 24448.0MB
  Temperature            :
    DDR0                 : 31.9 C
    DDR2                 : 30.5 C
    DDR4                 : 28.6 C
    DDR5                 : 29.2 C
    Core0                : 28.9 C
    Core1                : 28.9 C
  Board_Power            : 6.42 W
--------------------------------------------------------------------------------
```

当能检测到安装的每个后摩模块时，则证明驱动安装完成，启动正常。

驱动安装完成后，后续使用分为两条路径：

| 路径 | 说明 |
| --- | --- |
| [路径一：依赖 Docker 镜像](#路径一依赖-docker-镜像) | 使用 Houmo 官方 Docker 镜像，完整工具链与环境变量均已预置，开箱即用 |
| [路径二：裸板（不依赖 Docker）](#路径二裸板不依赖-docker) | 不依赖 Docker，在板端 Python 环境（建议 venv）pip 安装 runtime 与 hmatc，仅做推理验证 |

### 路径一：依赖 Docker 镜像

Docker 镜像是 Houmo 官方完整工具链环境（Ubuntu 24.04 + Python 3.12），hmatc、tcim_lite、modelscope 等已预装在镜像内（/opt/venv/dadao），HOUMO_TARGET / HOUMO_VERSION / TCIM_* 等环境变量也已预置，无需再 pip 安装任何依赖、无需 export。

#### 安装 Docker 镜像

执行如下指令

```bash
sudo apt update
sudo apt install docker.io
sudo docker load -i Dadao-docker-xh2-v1.2.0-ubuntu20.04-aarch64.tar
```

> 普通用户不在 docker 组，直接执行 `docker` 命令会报 `/var/run/docker.sock` 权限拒绝，本文统一加 `sudo`。如需免 sudo：`sudo usermod -aG docker <用户名>` 后重新登录生效。

> **Docker 安装异常**
>
> ```
> Loaded image: harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64
> Error unpacking image harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64: apply layer error for "harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64": failed to extract layer sha256:b4c711080c6c7c55fedf111aa657a7799dd3bd28aed19b017a12266c6fcb4dcc: failed to convert whiteout file "root/.pip/.wh..wh..opq": operation not supported
> ```
>
> 安装镜像时，docker报错如上。报错是由于我们的 Overlay Rootfs冲突，解决方法就是用真实的 ext4 不用 overlay，固件里只是软连接还不够。
>
> ```bash
> systemctl stop docker
> rm -rf /userdata/docker/
> mkdir -p /userdata/docker_real
> chmod 711 /userdata/docker_real
> mkdir -p /etc/docker
> cat > /etc/docker/daemon.json <<'EOF'
> {
>   "data-root": "/userdata/docker_real",
>   "storage-driver": "overlay2"
> }
> EOF
> systemctl start docker
> ```



#### 模块测试

下载`houmo-examples-xh2_v1.2.0.zip`后，运行如下指令。

```
unzip houmo-examples-xh2_v1.2.0.zip
cd houmo-examples-xh2/
sudo docker run -it --name Dadao-xh2-1.2.0 --pid=host --privileged --shm-size 64g -v "$PWD:/workspace" -w /workspace harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64 /bin/bash
source env.sh

# 测试 ddr 带宽
/workspace/tools/bandwidth_perf# ./run.sh

# 测试 pcie 带宽
/usr/local/houmo-sdk/hal/utility/pcie_test 0

# 测试算力性能
/workspace/tools/computing_perf# ./run.sh
```

#### 模型测试（镜像内）

```bash
cd houmo-examples-xh2/
sudo docker run -it --name Dadao-deploy-xh2 --pid=host --privileged --shm-size 64g \
  -v "$PWD:/workspace" -w /workspace \
  harbor.houmo.ai/toolchain/release:Dadao-deploy-xh2-v1.4.0-ubuntu24.04-aarch64 /bin/bash

source env.sh
cd models/llm/qwen3.5

# 容器内预装 modelscope，tokenizer 自动下载，无需手动拷 hf_config
python3 get_model.py --type hmm --model_name qwen3.5 --model_size 0.8b

# demo 默认图片不在 examples 包内，先放一张到默认路径（原因见路径二「模型测试」小节的注意）
mkdir -p /workspace/data/pic && cp /workspace/hmodel/xh2/examples/llm/qwen3omni/data/cars.jpg /workspace/data/pic/beach.jpeg
python3 demo.py --model_name qwen3.5 --model_size 0.8b
```

镜像 tag 以实际下载的 tar 为准（`sudo docker images` 查看）；9b 等大模型需先给宿主机加 swap（同[路径二：裸板](#路径二裸板不依赖-docker)的「内存不足时加 swap」小节，swap 对容器同样生效）。

### 路径二：裸板（不依赖 Docker）

以下以 v1.4.0 资源为例，请按实际版本替换文件名中的版本号。裸板仅做推理验证：AArch64 不支持模型量化与编译（需要 x86 + CUDA 环境），请使用编译好的 .hmm 模型。

#### 配置基础环境

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip build-essential python3-dev

# pip 换国内源（华为云/阿里实测较快；清华源在部分网络下 403 不可用）
pip3 config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple
pip3 config set global.extra-index-url https://mirrors.aliyun.com/pypi/simple/
```

#### 安装 runtime（pip 安装，不要解压到 /opt）

runtime 压缩包是标准 Python 源码包（sdist），直接 pip 安装，会为当前 Python 版本现场编译 tcim_lite：

```bash
cd houmo-examples-xh2
python3 -m venv .venv
source .venv/bin/activate
pip3 install houmo_tcim_runtime_xh2_linux_aarch64-1.4.0.tar.gz
python3 -c "import tcim_lite; print(tcim_lite.__file__)"    # 验证
```


#### 安装 hmatc 工具链

```bash
source /etc/profile.d/houmo-sdk.sh    # 提供 HOUMO_SDK_PATH（hmatc 编译扩展时要找 HAL 头文件；非登录 shell 不会自动加载）
pip3 install "torch==2.8.0" "torchvision==0.23.0"   
pip3 install -r hmatc/requirements.txt
pip3 install requests tqdm loguru onnx-graphsurgeon opencv-python-headless    # requirements 漏列的依赖
export HOUMO_TARGET=xh2        # hmatc 的 setup.py 强制要求
cd hmatc && ./install.sh
cd ..
python3 -c "import hmatc; print(hmatc.__file__)"    # 验证
```

#### 设置环境变量

| 变量 | 值 | 说明 |
| ---- | -- | ---- |
| HOUMO_TARGET | xh2 | 必需。demo / build / get_model 用它拼接模型路径，未设置直接报错 |
| HOUMO_VERSION | v1.4.0 | get_model 需要（写入模型版本信息），格式 vX.Y.Z |
| TCIM_FORCE_BACKEND | Xh2HalBackend | 推理必需。未设置会报 No available devices |
| HOUMO_SDK_PATH | /usr/local/houmo-sdk | 安装 hmatc（编译扩展）时需要。`source /etc/profile.d/houmo-sdk.sh` 获得，非登录 shell 不会自动加载 |
| TCIM_RUNTIME_PATH、PYTHONPATH | — | 仅旧 /opt 流程需要，pip 安装 runtime 后无需设置 |

注意：仓库根目录的 env.sh 只负责补 PATH / LD_LIBRARY_PATH 等路径，**不会设置**上表变量（Docker 镜像内为预置环境），裸板需先自行 export：

```bash
export HOUMO_TARGET=xh2 HOUMO_VERSION=v1.4.0 TCIM_FORCE_BACKEND=Xh2HalBackend
source env.sh    # 可选，仅用于补路径
```

也可写入系统配置：

```bash
echo 'export HOUMO_TARGET=xh2 HOUMO_VERSION=v1.4.0 TCIM_FORCE_BACKEND=Xh2HalBackend' | sudo tee /etc/profile.d/houmo-env.sh
```

#### 模型测试（裸板）

qwen3.5 还有 0.8b / 2b / 4b 规格（见该目录 README.MD）。快速验证建议先跑 `--model_size 0.8b`：模型仅约 2.1 GB、加载峰值内存低，8 GB 内存的板子无需加 swap 即可运行（实测 E2E 约 25 tokens/s）。

```bash
cd models/llm/qwen3.5
pip3 install transformers==5.5.0 onnx-ir==0.1.13    # demo 依赖；勿用 -r requirements.txt（其内嵌清华源在部分网络 403）

# 只拉预编译 .hmm，务必带 --type hmm 与模型参数；不带会继续去拉全量原始权重（裸板用不上，且缺 modelscope 包会报错）
python3 get_model.py --type hmm --model_name qwen3.5 --model_size 9b
# hmm 全部就位后，结尾 tokenizer 阶段仍会报一次 ModuleNotFoundError: modelscope —— 属预期，忽略即可

mkdir -p Qwen3.5-9B && cp -a output/xh2/hmquant/hf_config/. Qwen3.5-9B/

# demo 默认图片不在 examples 包内，先放一张图到默认路径（--image_path 参数因脚本缺陷只能追加、无法替换默认值）
mkdir -p ../../data/pic && cp ../../hmodel/xh2/examples/llm/qwen3omni/data/cars.jpg ../../data/pic/beach.jpeg

python3 demo.py --model_name qwen3.5 --model_size 9b
```

> qwen3.5 是多模态模型，非交互模式下固定提问“描述这些图片”，`--question` 参数不生效；描述对象就是上面放置的 `$HOUMO_EXAMPLES_PATH/data/pic/beach.jpeg` 那张图（需先 `source env.sh` 让变量生效）。
>
> 更多模型与使用方法，请参考 models 目录下其他模型文件夹下的 README。

#### 内存不足时加 swap

9b 级模型加载峰值内存超过 8GB，8GB 内存的板子会触发内核 OOM（dmesg 可见 `Killed process ... python3`）。先加 swap 再运行：

```bash
sudo fallocate -l 8G /userdata/swapfile
sudo chmod 600 /userdata/swapfile
sudo mkswap /userdata/swapfile
sudo swapon /userdata/swapfile
echo '/userdata/swapfile none swap sw,nofail 0 0' | sudo tee -a /etc/fstab
```

### hm_smi

可以通过hm_smi 工具控制卡的功耗和频率，下⾯是详细参数说明

| **参数**           | **说明**                                                     | **备注**                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -g, --gov          | 设置DVFS采取的策略                                           | 当前⽀持三种策略performance:最⼤性能,不调频,ipu跑最⾼频率,最⾼频默认为1400Mhz,如果使⽤lc修改,则为lc修改后的值<br>ondemand:根据ipu的负载,动态调整ipu的运⾏频率<br>powerlimit:根据⽤⼾指定的功耗,动态调整ipu的频率.(从⽀持的最低频率开始,需要smi⼯具⼀直运⾏监控)(powerlimit模式需要smi⼯具⼀直运⾏监控,⼀旦退出,会切到ondemand模式,直到下次-g命令修改策略) |
| -pl, --powerlimit  | 采⽤powerlimit模式时的必选参数，需要另外指定最⼤功耗,也可以继续指定最⼩功耗（单位为W） | 当ipu以最低频率运⾏,依然超过设置的最⼤功耗，会维持在最低频率当ipu以最⾼频率运⾏,依然低于设置的最低功耗,会维持在最⾼频率 |
| -lc, --lock_clocks | 锁定的ipu的最⼤频率,也可以继续指定最⼩频率(单位为Mhz)        | 限制ipu运⾏的频率范围,设置后⼀直⽣效,直到下⼀次使⽤lc修改与governor策略并⾏,共同起作⽤ |

**使用示例**

| 指令                                                         | 解释                                                         | 备注                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| hm_smi -g ondemand                                           | 配置当前模式为ondemand                                       |                                                        |
| hm_smi -g performance                                        | 配置当前模式为performance                                     |                                                        |
| hm_smi -g powerlimit -pl 10w                                 | 配置当前模式为power_limit，⽽且最⼤功耗为10W                 |                                                        |
| hm_smi -g powerlimit -pl 10w 6.5w                            | 配置当前模式为power_limit，允许功耗在10W~6.5W之间波动        | 功耗在10W~6.5W之间波动的时候不再调整IPU频率            |
| hm_smi -a -lc 1000 500 -g powerlimit -pl 5w 3w               | 配置当前模式为power_limit，允许功耗在5W~3W之间波动,ipu最⼤频率为1000Mhz,最⼩频率为500Mhz | ipu频率可⽤档位:1400/1200/1000/850/700/500/200         |
| `hm_smi -lc 1000 500` 等同于 `hm_smi -g performance -lc 1000 500` | 配置当前模式为performance模式,ipu运⾏频率范围最⼤为1000M,最低为500M. | 实际ipu运⾏在1000Mhz,因为performance模式是运⾏最⾼频率 |



更多 hm_smi 指令，参考[M50 SMI工具使用指南 — M50 SMI工具使用指南 1.0.0 文档](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/system-installation-device-management/device-monitoring-and-management/smi_tool_user_guide/index.html)





## 模型编译

请参考[后摩大道® M50 TCIM用户手册 — M50 TCIM用户手册 1.4.0 文档](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/model-build-and-inference/tcim_user_guide/index.html)







## FAQ

### PCIe的通道lane数不同

不同的板子的PCIe接口可能不相同，Houmo模块最大支持4lane的通道，会根据PCIe的可用通道数上电自动切换。从测试结果来看，LLM，VLM这一类推理数据比较小的场景，PCIe 带宽对推理性能几乎没有影响，只对模型第一次加载速度有一定影响。PCIe 带宽主要影响 CV 类像 Yolo 、图像识别之类数据交流比较频繁的应用。



### 裸板使用 hmatc / tcim_lite 报 ModuleNotFoundError

runtime 压缩包内的 tcim_lite 预编译扩展只适配 Python 3.9（Docker 镜像环境），解压到 /opt 再设 PYTHONPATH 的方式在 Debian 12（Python 3.11）等环境下无法导入；hmatc 也不随 runtime 分发。正确做法是在目标 Python 环境（建议 venv）内 pip 安装 runtime 源码包，现场编译适配当前 Python 版本：

```bash
pip3 install houmo_tcim_runtime_xh2_linux_aarch64-<version>.tar.gz
```

纯二进制测试工具（tcim_perf 等）不涉及 Python，只需为其设置 runtime lib 目录的 LD_LIBRARY_PATH。
### 下载速度太慢

建议更换国内镜像源。

pip 更换国内源（任选其一）：

```bash
pip3 config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple   # 华为云
pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/               # 阿里云
pip3 config set global.index-url https://mirrors.ustc.edu.cn/pypi/simple               # 中科大
pip3 config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple         # 腾讯云
```

apt 源同理（Debian 12，把域名换成上面任意一家）：

```bash
sudo sed -i 's|deb.debian.org|mirrors.ustc.edu.cn|g; s|security.debian.org|mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sudo apt update
```

注意：清华源（tuna）在部分网络环境下返回 403 不可用；examples 内个别 requirements.txt 写死了 tuna 作为 extra-index，遇到下载失败时可改为直接指定包名安装。