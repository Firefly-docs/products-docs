# Compile Android7.1 Industry firmware


## Download compilation instructions:
Android7.1 industry version it is more widely used in industry and tablet and box fields, and has been verified for stable performance in mass production.
This version also serves as our main maintenance version and is applicable to all models of our RK3399 system.


## Compile Android7.1 industry firmware

### Download Android SDK

**The Android SDK source package is relatively large, you can go to the download page to get the Android7.1 source package:**
[Android7.1 industry SDK]

After downloading, verify the MD5 code:
```
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.001
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.002
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.003
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.004

3387ebaa3d8e43dd1164527d4054621b  firefly_rk3399_industry7.1_git_20211216.7z.001
a74bb71622add3d2b558dc5a0bfb51e4  firefly_rk3399_industry7.1_git_20211216.7z.002
00f3202d42559e02cefc5300e48e1690  firefly_rk3399_industry7.1_git_20211216.7z.003
fb64756ff7e24d9bb76ecd4678afb55f  firefly_rk3399_industry7.1_git_20211216.7z.004
```
After confirming that it is correct, you can unzip:
```
mkdir -p ~/proj/firefly-rk3399-Industry
cd ~/proj/firefly-rk3399-Industry
7z x ~/firefly_rk3399_industry7.1_git_20211216.7z.001 -r -o.
git reset --hard
```
Note: Be sure to update the remote warehouse after decompression.
The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/firefly-rk3399-Industry

2. Download remote bundle repository
git clone https://gitlab.com/TeeFirefly/rk3399-industry-nougat-bundle.git .bundle

3. If the download warehouse fails, the current bundle warehouse is about 1.4G, so there may be stuck or failed problems during synchronization. You can download and unzip it from the Baidu cloud link below to the SDK root directory.

7z x rk3399-industry-nougat-bundle.7z  -r -o. && mv rk3399-industry-nougat-bundle/ .bundle/

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD
```

Google Driver[[bundle download]](http://en.t-firefly.com/doc/download/78.html#other_230)




## EC-A3399C industry version product compilation method


## Other Android versions
* <font color=#ff0000 size=3>Main maintenance:</font>

   ["Compile Android7.1 Industry firmware"](compile_android7.1_industry_firmware.md)  
* Support but not maintain:

   ["Compile Android7.1 firmware"](compile_android7.1_firmware.md)     ["Compile Android8.1"](compile_android8.1_firmware.md)  

[Android7.1 industry SDK]: https://en.t-firefly.com/doc/download/55.html#other_230
