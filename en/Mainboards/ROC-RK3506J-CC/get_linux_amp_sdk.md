## Get SDK

First, prepare an empty folder to store the SDK. It is recommended to be in the home directory. This article takes `~/proj` as an example

<font color=red>

**Note:**

**1. The SDK uses cross-compilation, so use the SDK on an X86_64 computer and do not download the SDK to the board**

**2. Please use Ubuntu22.04 (real machine or docker container) as the compilation environment. If you use other versions, compilation errors may occur**

**3. Do not store or decompress the SDK in a shared folder of the virtual machine or a non-English directory**

**4. Please use a normal user throughout the process of obtaining and compiling the SDK. Root permissions are not allowed or required (unless apt is required to install the software)**
</font>

### Installation tools
To obtain the SDK, you need to install:
```
sudo apt update
sudo apt install -y repo git python p7zip-full
```
### Initialize the warehouse

**We provide a basic SDK package. Please contact sales@t-firefly.com to obtain it. **

After downloading, verify the MD5 code first:

```bash
$ md5sum rk3506_linux_release_20250117_v1.0.0a/*
a5df458069569e9288b1d96097b43397  rk3506_linux_release_20250117_v1.0.0a.7z.001
7c1a1b050ae4f0daf64fe7488a7f4b02  rk3506_linux_release_20250117_v1.0.0a.7z.002
c7998713e7ef3512635ad2f79730d888  rk3506_linux_release_20250117_v1.0.0a.7z.003
```

After confirmation, you can decompress it:
```bash
mkdir -p ~/proj/rk3506_sdk
cd ~/proj/rk3506_sdk
7z x /path/to/rk3506_linux_release_20250117_v1.0.0a/rk3506_linux_release_20250117_v1.0.0a.7z.001
```