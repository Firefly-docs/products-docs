# ADB 使用

## 简介 [step]

ADB（Android Debug Bridge）是 Android 的命令行调试工具，可用于查看系统日志、上传下载文件、安装应用等操作。Firefly-R3572A 提供 1 × Type-C（USB2.0 OTG）作为 ADB 连接口，也支持网络 ADB。

## 准备连接 [step]

使用 Type-C 数据线连接电脑与主板 OTG 口，OTG 口位置见 [接口描述](../product_introduction/started.md)。

在 Android 系统中依次打开「设置」→「已连接的设备」→「Connect to PC」，连接后电脑端执行以下命令确认设备已识别：

```bash
adb devices
```

预期输出中设备状态为 `device`。若显示 `unauthorized`，在主板弹出的授权对话框中确认允许调试。

## 网络 ADB [step]

主板与电脑处于同一局域网时，可以通过网络使用 ADB，无需数据线：

1. 在「开发者选项」中打开「网络 ADB 调试」。
2. 查看主板 IP 地址，例如通过「设置」→「关于设备」→「状态信息」。
3. 在电脑端执行：

```bash
adb connect <主板 IP 地址>:5555
adb shell
```

## 常用命令 [step]

| 命令 | 作用 |
| --- | --- |
| `adb devices` | 列出已连接设备 |
| `adb shell` | 进入主板命令行 |
| `adb logcat` | 查看系统日志 |
| `adb push <本地路径> <设备路径>` | 上传文件到主板 |
| `adb pull <设备路径> <本地路径>` | 从主板下载文件 |
| `adb install <apk>` | 安装应用 |
| `adb reboot` | 重启主板 |
| `adb reboot loader` | 重启进入 Loader 升级模式 |

### 连接管理

```bash
adb devices                       # 列出已连接设备
adb tcpip 5555                    # 让设备端的 adbd 重启，并在 TCP 端口 5555 处监听
                                  # 此时可以断开 USB 连接
adb connect 192.168.1.100:5555    # 远程连接设备，设备的 IP 地址是 192.168.1.100
adb disconnect                    # 断开连接
```

## 调试 [step]

### 获取系统日志 adb logcat

```bash
adb logcat                        # 查看全部日志
adb logcat -s tag                 # 仅查看部分日志
```

### 运行命令 adb shell

```bash
adb shell                         # 进入设备的命令行
adb shell <命令>                  # 直接执行单条命令
```

#### 获取详细运行信息 adb bugreport

```bash
adb bugreport                     # 生成详细运行信息报告
adb bugreport report.zip          # 保存到本地，方便用编辑器查看
```

### root 权限

```bash
adb root                          # 以 root 权限重启 adbd（eng/userdebug 固件）
adb remount                       # 重新挂载 system 分区为可写
```

## 应用管理 [step]

### 安装应用 adb install

```bash
adb install facebook.apk          # 安装 facebook.apk
adb install -r twitter.apk        # 升级 twitter.apk
```

### 卸载应用 adb uninstall

```bash
adb uninstall com.example.app     # 卸载应用（按包名）
adb uninstall -k com.example.app  # 卸载应用但保留数据与缓存目录
```

### 命令行帮助信息 adb help

```bash
adb help                          # 查看 adb 支持的命令行帮助信息
```

## Windows 上安装 ADB [step]

1. 参照 [安装 RK USB 驱动](../getting_started/firmware_upgrade/upgrade_firmware.md) 完成驱动安装。
2. 下载 Android SDK Platform-Tools 并解压，例如解压到 `C:\platform-tools`。
3. 打开命令行窗口，切换到解压目录后执行：

```text
cd C:\platform-tools
adb devices
```

## Ubuntu 上安装 ADB [step]

```bash
sudo apt-get install android-tools-adb android-tools-fastboot
adb devices
```