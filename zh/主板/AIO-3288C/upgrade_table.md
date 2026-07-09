# 烧写须知

由于不同系统的固件存在差异，烧写固件时可能会存在一些问题。若遇到烧写问题，可按照如下操作。

## 准备工作

### 固件

* **Linux**

    Firefly 发布的 Linux 固件采用 GPT 分区格式管理启动介质，早期采用 <font color=#ff0000> **MBR 分区格式的固件已经停止支持** </font>。

* **Android 5.1**

### 烧写工具

* **[Upgrade_tool](http://www.t-firefly.com/doc/download/51.html#linux_22)**
* **[Android_tool](http://www.t-firefly.com/doc/download/51.html#windows_22)**

## 烧写 Android 固件须知

烧写 Android 固件时，请仔细阅读表格：

![](../../../rk3288_img/load_android_notice.png)

重点说明：在 Windows 下，Linux 升级 Android5.1 时，如何进入 Maskrom 模式：

**步骤如下:**

1. 用 AndroidTool_v2.58 工具，点击 “擦除flash”，这步的作用是 “擦除IDB”，但无法擦除 flash，因此会出现 “擦除 flash 失败” 的情况。
2. 重启板子，用 AndroidTool_v2.35 工具，点击 “擦除flash”，现在才是真正的 “擦除flash”。
3. 点击 “升级”，开始烧写 Android5.1 固件。

**提示：若有用户在使用 AndroidTool_v2.35 工具时，点击 “擦除flash” 和 “升级” 都会出现 “测试设备失败” 的情况下，换 AndroidTool_v2.47 版本工具。**

## 烧写 Linux-GPT 固件须知

烧写 Linux-GPT 固件时，请仔细阅读表格：

![](../../../rk3288_img/load_linux_gpt_notice.png)

重点说明：

一、在 Windows 下，Linux-MBR 或 Android5.1 升级 Linux-GPT，如何进入 Maskrom 模式：

**步骤如下：**

1. 用 AndroidTool_v2.58 工具，点击 “擦除flash”，完成后，设备自动进入 Maskrom 模式。
2. 点击 “升级”，开始烧写 Linux-GPT 固件

二、在 Linux 下，Linux-MBR 或 Android5.1 升级 Linux-GPT，如何进入 Maskrom 模式：

**步骤如下：**

```bash
# 用 upgrade_tool_v1.24 工具，要进行擦除 flash，则运行：
sudo upgrade_tool_v1.24 ef /path/to/Linux-GPT

# 上述完成后，用 upgrade_tool_v1.34 工具，烧写 Linux-GPT 固件，运行如下：
sudo upgrade_tool_v1.34 uf /path/to/Linux-GPT
```

## 烧写 Linux-MBR 固件须知

烧写 Linu-MBR 固件时，请仔细阅读表格：

![](../../../rk3288_img/load_linux_mbr_notice.png)

在上述表格中，都是进入 Loader 模式下，正常升级。