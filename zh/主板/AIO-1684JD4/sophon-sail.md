# Sophon Sail

Sophon Sail 对 SophonSDK 中的 BMLib、BMDecoder、BMCV、BMRuntime 进行了封装，将 SophonSDK 中原有的“加载 bmodel 并驱动 TPU 推理”、“驱动 TPU 做图像处理”、“驱动 VPU 做图像和视频解码”等功能抽象成更为简单的 C++ 接口对外提供；并且使用 pybind11 再次封装，提供简洁易用的 python 接口。

如果想使用 Sophon Sail 需要参考[Sophon SDK 开发](sophon-sdk.md)章节中的文档编译 Sophon Sail 安装包。

[下载中心]可直接获取已经编译好的 Sophon Sail 安装包，安装方法如下：

```bash
pip3 install sophon_arm-3.7.0-py3-none-any.whl
```

验证是否正常安装：

```bash
linaro@aibox-1684x:~$ python3
Python 3.8.2 (default, Mar 13 2020, 10:14:16)
[GCC 9.2.1 20200306] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sophon.sail as sail
>>> sail.get_available_tpu_num()
1
>>>
```

[下载中心]: https://www.t-firefly.com/doc/download/179.html