# Houmo LQ50 M.2 AI Computing Module

## Product Introduction

### Overview



| Front | Back |
| :---: | :---: |
| ![](../../../modules_img/HOUMO-LQ50/houmo-lq50-1.png) | ![](../../../modules_img/HOUMO-LQ50/houmo-lq50-2.png) |

### Specifications

Please contact sales (sales@t-firefly.com) to obtain the datasheet of the corresponding module.



## Usage

Considering that the Houmo module resources are mainly tested with Docker containers and are large in size, and the driver is automatically compiled and installed via DKMS, which requires network access and a complete development environment on the board for the first installation, Firefly releases resources in two forms. One is the Firefly test package: based on our own firmware, with the prebuilt driver and test tools packaged together, so that customers using Firefly firmware can test directly (not maintained long-term, mainly used to check whether the module works properly). The other is the official Houmo resource package. After completing the preliminary test of the Houmo module, customers can download the corresponding resources according to their needs.



### Firefly Houmo Test Package

The package includes:

1. Prebuilt driver of the Houmo LQ50 module for the aarch64 platform
2. Test package
3. Runtime archive
4. Installation script



Run the installation script and it will automatically install the driver and place the runtime package under the `/opt/` directory. Decompress the test package and run `run_all.sh` to start the test. By default only the check test runs, which is used to verify that the module and the environment work properly. You can run `run_all.sh --aging` for a 2-hour stress test. It tests the DDR read/write speed, computing power, and inference performance of the module respectively (the inference performance verification uses a CNN model, mainly to verify the functional integrity of the module).



### Resource Package Download

Please contact sales (sales@t-firefly.com) to get the **Houmo cloud drive resources** download link, and select the corresponding Houmo version to download according to your needs. The Houmo resources uploaded on the cloud drive have all been verified on the firmware we provide (e.g. Debian 12).



### Environment Setup

**Download resources**

```
houmo-drv-<target_hw>_<release>_${distro}_$arch.run			# Driver package
Dadao-deploy-docker-xh2-vx.y.z-<rootfs>-aarch64.tar			# Test environment image
houmo-examples-xh2_<version>.zip		# Test program package
```

For a detailed introduction to the package naming, refer to [3.1. Linux Host Installation and Deployment — M50 Software Platform Quick Start 1.4.0](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/quickstart/getting-started/quickstart_guide/setup/linux.html) and [3.2. Ubuntu/Kylin V11/UOS Driver Installation and Uninstallation — M50 Software Platform Driver Installation Guide 1.4.0](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/system-installation-device-management/environment-deployment/system_software_installation_guide/linux/ubuntu.html).

The following tutorial uses version V1.2.0 as an example. Replace it according to your needs.

**Install the driver**

```bash
# Install the PCIe driver of the accelerator card
apt  update
sudo apt-get install python3 python3-dev python3-pip -y
apt install /boot/linux-headers-6.1-arm64_arm64.deb
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
chmod a+x houmo-drv-xh2_v1.2.0_linux_aarch64.run
sudo bash ./houmo-drv-xh2_v1.2.0_linux_aarch64.run --build-host false install all
```

> **Error: linux-headers-6.1 not found**
>
> ```
> cd /boot
> ls
> ```
>
> Check the version corresponding to the board
>
> Run the following command
>
> ```bash
> dpkg -i /boot/linux-headers-<version>-arm64_arm64.deb # Replace it with the deb name of the corresponding version
> ```
>
> Run again
>
> ```bash
> sudo bash ./houmo-drv-xh2_v1.2.0_linux_aarch64.run --build-host false install all
> ```

After the installation is complete, run:

```bash
hm_smi -a
```

You should see output similar to the following:

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

When every installed Houmo module can be detected, it indicates that the driver is installed and started properly.



**Install the Docker image**

Run the following commands:

```bash
sudo apt update
sudo apt install docker.io
docker load -i Dadao-docker-xh2-v1.2.0-ubuntu20.04-aarch64.tar
```

> **Docker installation error**
>
> ```
> Loaded image: harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64
> Error unpacking image harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64: apply layer error for "harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64": failed to extract layer sha256:b4c711080c6c7c55fedf111aa657a7799dd3bd28aed19b017a12266c6fcb4dcc: failed to convert whiteout file "root/.pip/.wh..wh..opq": operation not supported
> ```
>
> When installing the image, docker reports the error above. The error is caused by a conflict with our Overlay rootfs. The solution is to use a real ext4 instead of overlay; a soft link in the firmware is not enough.
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



### Module Test

```
unzip houmo-examples-xh2_v1.2.0.zip
cd houmo-examples-xh2/
docker run -it --name Dadao-xh2-1.2.0 --pid=host --privileged --shm-size 64g -v "$PWD:/workspace" -w /workspace harbor.houmo.ai/toolchain/release:Dadao-xh2-v1.2.0-ubuntu20.04-aarch64 /bin/bash
source env.sh

# Test DDR bandwidth
/workspace/tools/bandwidth_perf# ./run.sh

# Test PCIe bandwidth
/usr/local/houmo-sdk/hal/utility/pcie_test 0

# Test computing performance
/workspace/tools/computing_perf# ./run.sh
```



### Model Test

Enter the `houmo-examples-xh2` decompressed in **Module Test**:

```
# If venv support is not installed
apt update
apt install -y python3-venv python3-pip

# Create a virtual environment in the project root (recommended)
cd /home/firefly/houmo-examples-xh2
python3 -m venv .venv

# Activate it (the current shell enters the venv)
source .venv/bin/activate

# Verify
which python3
pip -V

# Go back to the example directory and install dependencies
cd /home/firefly/houmo-examples-xh2/models/llm/qwen3.5
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
python3 -m pip install -r ../../../hmodel/gptqmodel/requirements.txt

# Run
python3 get_model.py --model_name qwen3.5 --model_size 9b
# --type raw: AArch64 does not support model quantization and compilation. Users need to use the compiled binary model files (.hmm or .hmms) to run inference on AArch64. Therefore the raw model is not pulled here.
```

> For more models and usage, refer to the README in the other model folders under /home/firefly/houmo-examples-xh2/models



### hm_smi

The hm_smi tool can be used to control the power consumption and frequency of the card. The following is a detailed description of the parameters:

| **Parameter**           | **Description**                                                     | **Notes**                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -g, --gov          | Sets the DVFS governor policy. Three policies are currently supported:<br>performance: maximum performance, no frequency scaling, the IPU runs at the highest frequency, which is 1400 MHz by default; if changed with lc, the value after the lc change applies<br>ondemand: dynamically adjusts the IPU operating frequency according to the IPU load<br>powerlimit: dynamically adjusts the IPU frequency according to the power consumption specified by the user (starting from the lowest supported frequency; the smi tool needs to keep running and monitoring) (in powerlimit mode the smi tool must keep running and monitoring; once it exits, the mode switches to ondemand until the next -g command changes the policy) | |
| -pl, --powerlimit  | Required parameter when using powerlimit mode. Additionally specify the maximum power consumption; the minimum power consumption can also be specified (in W) | When the IPU runs at the lowest frequency and still exceeds the set maximum power, it stays at the lowest frequency. When the IPU runs at the highest frequency and still stays below the set minimum power, it stays at the highest frequency |
| -lc, --lock_clocks | Locks the maximum IPU frequency; the minimum frequency can also be specified (in MHz) | Restricts the operating frequency range of the IPU. Once set, it remains in effect until the next lc change. It works in parallel with the governor policy |

**Usage examples**

| Command                                                         | Explanation                                                         | Notes                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| hm_smi -g ondemand                                           | Set the current mode to ondemand                                       |                                                        |
| hm_smi -g performance                                        | Set the current mode to performance                                     |                                                        |
| hm_smi -g powerlimit -pl 10w                                 | Set the current mode to powerlimit, with a maximum power of 10 W                 |                                                        |
| hm_smi -g powerlimit -pl 10w 6.5w                            | Set the current mode to powerlimit, allowing the power to fluctuate between 10 W and 6.5 W        | When the power fluctuates between 10 W and 6.5 W, the IPU frequency is no longer adjusted            |
| hm_smi -a -lc 1000 500 -g powerlimit -pl 5w 3w               | Set the current mode to powerlimit, allowing the power to fluctuate between 5 W and 3 W, with the IPU maximum frequency at 1000 MHz and the minimum frequency at 500 MHz | Available IPU frequency levels: 1400/1200/1000/850/700/500/200         |
| `hm_smi -lc 1000 500` is equivalent to `hm_smi -g performance -lc 1000 500` | Set the current mode to performance, with the IPU frequency range from 500 MHz (minimum) to 1000 MHz (maximum). | The IPU actually runs at 1000 MHz, because performance mode always runs at the highest frequency |



For more hm_smi commands, refer to [M50 SMI Tool User Guide 1.0.0](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/system-installation-device-management/device-monitoring-and-management/smi_tool_user_guide/index.html)





## Model Compilation

Please refer to the [Houmo Dadao® M50 TCIM User Manual 1.4.0](https://developer.houmoai.com/hmdoc/m50/software-manual/latest/model-build-and-inference/tcim_user_guide/index.html)







## FAQ

### Different PCIe lane counts

The PCIe interfaces of different boards may differ. The Houmo module supports up to 4 lanes and automatically switches according to the number of available PCIe lanes at power-on. According to the test results, for inference scenarios with relatively small data transfer such as LLM and VLM, PCIe bandwidth has almost no impact on inference performance, and only affects the speed of the first model loading to a certain extent. PCIe bandwidth mainly affects CV-type applications with frequent data exchange, such as Yolo and image recognition.



### pip install cannot find Houmo packages such as 'hmatc'

The environment inside the image is configured by default, so this problem does not occur there. If you do not rely on the image, but use the Firefly tool to automatically mount the Runtime package under /opt/, this problem is triggered. The reason is that the installer does not detect the package in the corresponding path.


```
# The name needs to be changed according to the version
export LD_LIBRARY_PATH=/opt/houmo_tcim_runtime_xh2_linux_aarch64-1.4.0/lib:/usr/local/houmo-sdk/hal/lib:$LD_LIBRARY_PATH
export TCIM_FORCE_BACKEND=Xh2HalBackend

```
Before executing the corresponding command, manually set the paths first.