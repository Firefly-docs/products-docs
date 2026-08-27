# Windows 驱动安装

## 安装驱动

我们的串口模块使用的是 CP2104，所以下载驱动并安装:

* [CP210X](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers)

如果您另外购买了使用其他芯片的模块，比如 CH340 或 PL2303，可以从这下载驱动：

* [CH340](https://sparks.gogo.co.nz/ch340.html)
* PL2303

如果在 Win8 上不能正常使用 PL2303，参考[这篇文章](https://blog.csdn.net/ropai/article/details/19619951)， 采用 3.3.5.122 或更老版本的旧驱动即可。

如果在 Windows 系统上安装官网的 CP210X 驱动，使用 PUTTY 或 SecureCRT 等工具设置串口波特率为 1500000，如果出现设置不了或无效的问题，可以下载旧版本[驱动](http://www.t-firefly.com/share/index/index/id/a2e8f25f3d53992bf3e04f45b0e6c8e8.html)。

插入适配器后，系统会提示发现新硬件，并初始化，之后可以在设备管理器找到对应的 COM 口：

![](../../../modules_img/USB-TO-TTL-Serial/debug_find_com.png)