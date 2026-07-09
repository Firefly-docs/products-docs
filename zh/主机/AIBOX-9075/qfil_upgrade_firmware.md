# 升级完整固件

## 进入 EDL 升级模式

### 硬件方式

1. 首先确认设备完全断开电源。

2. 使用 Type-A USB 数据线 连接设备的 TypeC 口和电脑。

3. 使用针或其他细小物品伸入恢复孔，按住里面的下载按钮，保持住。

4. 接入电源。

5. 2 秒后松开恢复按钮。

![](../../../qcom_img/AIBOX-9075/edl_key.jpg)
### 软件方式

在设备正常运行的情况下，使用 Type-A USB 数据线 连接好设备的烧录口和电脑，然后在设备终端或调试串口上运行：

```shell
sudo systemctl reboot edl
```

![](../../../qcom_img/AIBOX-9075/download_port.jpg)

### 确认是否成功

如果设备成功进入 EDL 模式，QFIL 工具上一般会显示 Qualcomm HS-USB QDLoader 9008。

![](../../../qcom_img/qfil_9008.jpg)

也有可能显示 Please Select an Existing Port，此时点击右侧 SelectPort，也可以看到 Qualcomm HS-USB QDLoader 9008，选中并点击 OK 即可。

![](../../../qcom_img/qfil_select_port.jpg)

**注意：必须要显示 9008 设备才行** 如果显示其他编号，如 900E，则说明设备状态异常，请断电重新尝试进入升级模式。

如果显示 No Port Available，则检查驱动是否安装成功、USB 线连接是否正确。

## 烧写固件

1 点击上方 Configuration，再点击 FireHose Configuration，在弹出的窗口中，按照下图设置。完成后点击 OK

![](../../../qcom_img/qfil_config.jpg)

2 在主界面选择 Flat Build，点击 Browse

![](../../../qcom_img/qfil_browse.jpg)

3 在弹出的窗口中前往你解压好的固件位置，文件类型选择 All Files，然后找到 xbl_s_devprg_ns.melf (也可能是 prog_firehose_ddr.elf，不同芯片不一样)并点击打开。

![](../../../qcom_img/qfil_open_melf.jpg)

4 主界面点击 Load XML，在弹出的串口中，全选所有 xml 文件并点击打开。此时又会弹出相同的窗口，继续选择全部 xml 文件点击打开。

![](../../../qcom_img/qfil_load_xml1.jpg)

![](../../../qcom_img/qfil_load_xml2.jpg)

5 最后点击 Download 开始烧录，等待它完成，需要几分钟。完成后如图

![](../../../qcom_img/qfil_download_finish.jpg)

6 完成后等待一会，设备应该会自动重启到正常模式。