## 获取 SDK

请联系销售 (sales@t-firefly.com) 获取 **RK182X SDK** 下载链接。

<font color=red>

**注意：**
<br>
**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**<br>
**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**<br>
**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**<br>
**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</font>

<br>
比如，SDK 压缩包是 `rk182x_linux_release_20260611_v1.0.4a.tgz`。

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf rk182x_linux_release_20260611_v1.0.4a.tgz
.repo/repo/repo sync -l
```

## 配置 
通过 `./build.sh config` 配置。

```
Select board type:
1) RK182X EVB1
2) RK182X SODIMM
3) RK182X SODIMM USB
4) RK182X M2
5) Cancel
#? 
```

选择 `2`

## 编译
```
./build.sh
```

生成的软件安装包在 `output/firmware/rknn3_rk182x_sodimm_installer_arm64.tgz`

## 安装
手动安装 RK1820/RK1828 软件包，按如下步骤操作：
* 拷贝 `rknn3_rk182x_sodimm_installer_arm64.tgz` 到 RK3588 或者 RK3576 端
* 解压 `tar xzf rknn3_rk182x_sodimm_installer_arm64.tgz`
* 安装 `./install.sh`
    * 安装重启后， RK3588 或者 RK3576 端系统会在启动后， ⾃动下载 RK182X 的固件，并启动后台服务程序。


## 其他
### 版本 V1.0.0
```
sudo rknn-smi -v
rknn-smi version : 1.1.0
PCIe driver version : 3.3.0
RC chips connect version : 3.2.0
EP chips connect version : 3.2.0
PCIe Device 0 firmware version: 1.0.0
rknn3 API version : 1.0.0
```
