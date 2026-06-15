### Download Firefly_Linux_SDK sub-volume compressed package

**Since the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of sub-volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source code package in the following ways：**[Firefly_Linux_SDK Source]

### Unpack Firefly_Linux_SDK sub-volume compressed package

The first time you use the SDK, you need to perform 3 steps. If you want to update the SDK later, you only need to perform step 3 to update the network.
```
1. Unpack the SDK

chmod +x ./sdk_tools.sh

mkdir ../firefly_sdk


Create a directory to store the SDK: For example, my current SDK is 3588, and I want to decompress it to the upper folder to avoid polluting the current directory

mkdir ../firefly_rk3588_SDK
./sdk_tools.sh --unpack -C ../firefly_rk3588_SDK


2. Restore the working directory

Select the directory you just decompressed

./sdk_tools.sh --sync -C ../firefly_rk3588_SDK


You can use the above script to execute or manually execute the command, choose one of them


# Enter the directory just after decompression, for example, here is ../firefly_rk3588_SDK
cd ../firefly_rk3588_SDK
.repo/repo/repo sync -l
.repo/repo/repo start firefly --all


3. Update the SDK

The first two steps are only performed when the SDK is decompressed for the first time, and the subsequent update of the SDK only needs to perform the third step for network update

.repo/repo/repo sync -c --no-tags

```

### Update Firefly_Linux_SDK

You can use the following command to update the SDK later
```
.repo/repo/repo sync -c --no-tags
```

[Firefly_Linux_SDK Source]: http://en.t-firefly.com/doc/download/60.html#other_358



