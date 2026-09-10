# 编译 RK1820/RK1828 安装包

## 获取 SDK

请联系销售 (sales@t-firefly.com) 获取 **RK182X SDK** 下载链接。

<font color=red>

**注意：**
<br>
**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**<br>
**2. 编译环境请使用 Ubuntu20.04或Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**<br>
**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**<br>
**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</font>

<br>
比如，SDK 压缩包是 `RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.X.X.tgz`。（具体版本以网盘名称最新发布为准）

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.X.X.tgz
.repo/repo/repo sync -l
```

### Bundle 更新
1.1.0a 后续版本将以 bundle 的形式更新，以减少下载时间。
下载并解压上述 SDK 基础包、完成同步后，将 bundle 包放置在 SDK 根目录下：

```
rk182x_sdk/
├── .repo/
└── bundle_xx_to_xx.tgz
```

解压对应的 bundle 包：

```
tar xzf bundle_xx_to_xx.tgz
```

在 SDK 根目录运行 bundle 内的脚本：
```
./bundle_xx_to_xx/bundle_update.sh
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

选择 `4`


```
Select Security Boot Mode:
1) Disable Secure Boot
2) Enable Secure Boot
3) Cancel
#? 
```

选择 `1`
> 该选项用于对固件进行相关安全加密，误操作可能导致固件后续无法升级，请谨慎选择。相关资料请参考1828SDK_Path/docs/Develop/Rockchip_RK1820_RK1828_User_Guide_SecureBoot_CN.pdf

## 编译
```
./build.sh
```

生成的软件安装包在 `output/firmware/rknn3_rk182x_m2_installer_arm64.tgz`

## 安装
手动安装 RK1820/RK1828 软件包，按如下步骤操作：
* 拷贝 `rknn3_rk182x_m2_installer_arm64.tgz` 到主控端
* 解压 `tar xzf rknn3_rk182x_m2_installer_arm64.tgz`
* 安装 `./install.sh`
    * 安装重启后， RK3588 或者 RK3576 端系统会在启动后， ⾃动下载 RK182X 的固件，并启动后台服务程序。


## 其他
### 版本 V 1.1.0
```
sudo rknn-smi -v
rknn-smi version              : 1.3.0
PCIe driver version           : 3.3.1
RC chips connect version      : 3.3.2
EP chips connect version      : 0.0.2
PCIe Device 0 firmware version: 1.1.0
rknn3 API version             : 1.1.0

```

## FAQ

### 当前支持的模型

当前支持的模型和详细说明，请参考 SDK 中的 `/home/zhang/rk182x_self/rknn/rknn3-runtime/doc/CN/00_RKNN3_SDK_发布说明_V1.1.0.pdf`。

### rknn3 API Version 显示 NA
安装一下binutils, 部分rootfs可能没有strings指令

```sh
sudo apt update
sudo apt install binutils
```