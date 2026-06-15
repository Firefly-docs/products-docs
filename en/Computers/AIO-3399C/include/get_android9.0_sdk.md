### Download Android SDK

Since the Android SDK source code package is relatively large, you can obtain the Android 9.0 source code package in the following ways:
[Download link](http://en.t-firefly.com/doc/download/52.html#other_247)

After the download is complete, verify the MD5 code before decompression：
```
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.001
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.002

6e1d3969c8a0f643522727ff07800bb5  rk3399pro_firefly_android9.0_20191126.7z.001
a6b8d6a775c3d5ed28f4d41cb210a84d  rk3399pro_firefly_android9.0_20191126.7z.002
```

Then unzip：
```
cd ~/proj/
7z x ./rk3399pro_firefly_android9.0_20191126.7z.001 -oAIO-3399Pro
cd ./AIO-3399Pro
git reset --hard
```



The following is how to update from gitlab：
```
#1. Enter the SDK root directory
cd ~/proj/AIO-3399Pro

#2. Download the remote bundle warehouse
git clone https://gitlab.com/TeeFirefly/rk3399pro-pie-bundle.git .bundle

#3. If downloading the warehouse fails, the current bundle warehouse occupies a large space, so there may be stuck or failed during synchronization.
# You can download it from the Baidu Cloud link below and unzip it to the SDK root directory. The unzip command is as follows:

7z x rk3399pro-pie-bundle.7z  -r -o. && mv rk3399pro-pie-bundle/ .bundle/

#4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

#5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD
```

You can also view the source code online at the following address:
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)
