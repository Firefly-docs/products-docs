# Sophon Sail

Sophon Sail encapsulated BMLib, BMDecoder, BMCV and BMRuntime in SophonSDK. The original functions of SophonSDK such as "loading bmodel and driving TPU inference", "driving TPU to do image processing", "driving VPU to do image and video decoding" are abstracted into a simpler C++ interface. It is also repackaged with pybind11 to provide a simple and easy-to-use python interface.

In order to use Sophon Sail, you need to compile the Sophon Sail installation package by referring to the documentation in the [Sophon SDK development](sophon-sdk.md) section.

The Sophon Sail installation package can be directly obtained from [Download Center]. The installation method is as follows:

```bash
pip3 install sophon_arm-3.7.0-py3-none-any.whl
```

Verify that the software is installed properly：

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

[Download Center]: https://en.t-firefly.com/doc/download/224.html