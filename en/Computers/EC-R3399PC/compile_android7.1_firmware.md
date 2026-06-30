**Download compilation instructions:**
At present, the Android7.1 version does not do major maintenance. Please compile and select the Android7.1 industry version<font color="#dd0000">(Main maintenance)</font>. This version is more widely used in the fields of industry and tablets and boxes. It has a stable performance and has been verified by mass production. This version As the main maintenance version of our company, it is applicable to all models of our RK3399 system.


# Compile Android7.1 firmware

**The Android SDK source package is relatively large, and the Android7.1 source package can be obtained as follows:**[Android7.1 SDK]

After downloading, verify the MD5 code:

```
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.001
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.002
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.003

90145747b33ad6834cd065b594f6687f  firefly_rk3399_android7.1_git_20211220.7z.001
6f28a1d66b1d168a89bfd3a7923775f8  firefly_rk3399_android7.1_git_20211220.7z.002
dc4d87ba8574900eed84f60f9cbe3646  firefly_rk3399_android7.1_git_20211220.7z.003
```

After confirming that it is correct, you can unzip:

```
mkdir -p ~/proj/ROC-RK3399-PC
cd ~/proj/ROC-RK3399-PC
7z x ~/firefly_rk3399_android7.1_git_20211220.7z.001 -r -o.
git reset --hard
```

* Update

```
#1.Enter root dir
cd ~/proj/ROC-RK3399-PC

#2. pull bundle.git first
git clone https://gitlab.com/TeeFirefly/rk3399-nougat-bundle.git .bundle

# 3.If clone fail, get bundle package from [Android7.1 SDK]
# The decompression instructions are as follows:

7z x rk3399-nougat-bundle-20190612.7z -r -o. && mv rk3399-nougat-bundle/ .bundle/

# 4.Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command

.bundle/update

# 5.Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD
```

## ROC-RK3399-PC Product compilation method



[Android7.1 SDK]: http://en.t-firefly.com/doc/download/51.html#other_18

