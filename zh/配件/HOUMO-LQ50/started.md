# 后摩 LQ50 AI 计算卡

## 产品介绍

### 产品简介



| 正面 | 反面 |
| :---: | :---: |
| ![](../../../modules_img/HOUMO-LQ50/houmo-lq50-1.png) | ![](../../../modules_img/HOUMO-LQ50/houmo-lq50-2.png) |

### 详细参数

请联系销售 (sales@t-firefly.com) 获取对应模块规格书。



## 使用方法

考虑到后摩模块资源主要以 docker 容器进行测试，体积较大。同时驱动安装为DKMS自动编译安装，需要板端第一次安装时有网络并配齐对应开发环境。因此 Firefly 发布两种形式的资源。一种是 Firefly 基于自身固件，将编译好的驱动和测试工具直接打包好，让使用 Firefly 固件的客户能直接测试（不进行长期维护，主要用于检测模块是否正常）。另一种则为 Houmo 官方资源包，完成初步对 Houmo模块的测试后，客户可以再根据需求进行下载对应的资源



### Firefly Houmo 测试包

内部包括

1. 编译好的Houmo LQ50 模块 aarch64 平台驱动
2. 测试包
3. runtime 压缩包
4. 安装脚本



运行安装脚本，会自动安装驱动，并将runtime包放置到 /opt/ 目录下。解压测试包，运行run_all.sh，即可进行测试，测试默认只运行 check 测试，用于检测模块与环境是否正常。可运行run_all.sh --aging 进行 2 hour 压力测试。会分别测试模块的DDR读写速率，算力， 以及推理性能（推理性能验证使用的是CNN模型，主要用于验证模块的功能完整）。



### 资源包下载

请联系销售 (sales@t-firefly.com) 获取 **Houmo 网盘资料** 下载链接，根据需求，选择相应 Houmo 版本进行下载。网盘中有上传的后摩资源，在我们所提供的固件上均已经过验证（如 Debian 12).



### 安装环境

**下载资源**

```
houmo-drv-<target_hw>_<release>_${distro}_$arch.run			# 驱动包
Dadao-deploy-docker-xh2-vx.y.z-<rootfs>-aarch64.tar			# 测试环境镜像
houmo-examples-xh2_<version>.zip		# 测试程序包
```

详细的包命名介绍，请参考[3.1. Linux主机端安装与部署 — M50 软件平台快速入门 1.4.0 文档](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/quickstart/getting-started/quickstart_guide/setup/linux.html) 与 [3.2. Ubuntu/Kylin V11/UOS驱动安装与卸载 — M50 软件平台驱动安装指南 1.4.0 文档](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/system-installation-device-management/environment-deployment/system_software_installation_guide/linux/ubuntu.html)

如下教程，都以V1.2.0版本举例，请根据需求替换。

**安装驱动**

```bash
# 安装加速卡 pcie 驱动
apt  update
sudo apt-get install python3 python3-dev python3-pip -y
apt install /boot/linux-headers-6.1-arm64_arm64.deb
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
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
  HMSW_Version   : V1.4.3
  HM_SMI_Version : V1.0.0
--------------------------------------------------------------------------------
  Wed Sep 16 05:35:51 CST 2026
--------------------------------------------------------------------------------
  device0 detail infos
--------------------------------------------------------------------------------
  Driver_Version         : V1.4.3
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



**安装 Docker 镜像**

执行如下指令

```bash
sudo apt update
sudo apt install docker.io
docker load -i Dadao-docker-xh2-v1.2.0-ubuntu20.04-aarch64.tar
```

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



### 模块测试

```
unzip houmo-examples-xh2_v1.2.0.zip
cd houmo-examples-xh2/
docker run -it --name Dadao-xh2-1.2.0 --pid=host --privileged --shm-size 64g -v "$PWD:/workspace" -w /workspace harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64 /bin/bash
source env.sh

# 测试 ddr 带宽
/workspace/tools/bandwidth_perf# ./run.sh

# 测试 pcie 带宽
/usr/local/houmo-sdk/hal/utility/pcie_test 0

# 测试算力性能
/workspace/tools/computing_perf# ./run.sh
```



### 模型测试

进入到我们**模块测试**中解压得到houmo-examples-xh2

```
# 如果没装 venv 支持
apt update
apt install -y python3-venv python3-pip

# 在项目根创建虚拟环境（推荐）
cd /home/firefly/houmo-examples-xh2
python3 -m venv .venv

# 激活（当前 shell 进入 venv）
source .venv/bin/activate

# 确认
which python3
pip -V

# 回到示例目录安装依赖
cd /home/firefly/houmo-examples-xh2/models/llm/qwen3.5
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
python3 -m pip install -r ../../../hmodel/gptqmodel/requirements.txt

# 运行
python3 get_model.py --model_name qwen3.5 --model_size 9b
# --type raw 由于AArch64架构不支持模型量化和编译操作。用户需要使用编译后二进制模型文件（.hmm或.hmms）在AArch64架构环境中推理。因此这里不拉取raw模型
```

> 更多模型与使用方法，请参考/home/firefly/houmo-examples-xh2/models目录下的其他模型文件夹下的Readme



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



### pip install 时找不到'hmatc'类似的 Houmo 包
镜像内部默认配置好的相关的环境，因此不会遇到此类问题。如果您是不依赖于镜像，而是通过 Firefly 的工具将 Runtime 包自动挂到了/opt/下，则触发此问题。原因是安装程序没有在对应路径检测到包


```
# 名称需要根据版本改动
export LD_LIBRARY_PATH=/opt/houmo_tcim_runtime_xh2_linux_aarch64-1.4.0/lib:/usr/local/houmo-sdk/hal/lib:$LD_LIBRARY_PATH
export TCIM_FORCE_BACKEND=Xh2HalBackend
```
在执行对应指令前，先手动设置下路径即可。