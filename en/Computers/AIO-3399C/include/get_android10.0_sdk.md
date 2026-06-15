### Download Android SDK

Due to the larger SDK, you can choose the cloud disk to download **Firefly-RK3399_Android10.0_git_20211222.7z** from the download page :

[Download link](http://en.t-firefly.com/doc/download/52.html#other_339)

Note: the source code and bundle are stored in the Google Driver


After downloading, verify the MD5 code:
```
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.001
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.002
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.003
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.004
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.005

fb41fcdc48b1cf90ecac4a5bb8fafc7a  Firefly-RK3399_Android10.0_git_20211222.7z.001
82d665fb54fb41245d9317312e7b7578  Firefly-RK3399_Android10.0_git_20211222.7z.002
0dd40827976200a6fb61623c9e87fe7c  Firefly-RK3399_Android10.0_git_20211222.7z.003
32324c3afe615394ee4cdf842c53ea8d  Firefly-RK3399_Android10.0_git_20211222.7z.004
9b1656ae07b4fe9ddf7d9ef2dfa2f95a  Firefly-RK3399_Android10.0_git_20211222.7z.005
```


After confirming that it is correct, you can unzip:
```
cd ~/proj/
7z x ./Firefly-RK3399_Android10.0_git_20211222.7z.001 -ork3399_Android10.0
cd ./rk3399_Android10.0
git reset --hard
```

Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/rk3399_Android10.0


2. Download remote bundle repository
git clone https://gitlab.com/TeeFirefly/rk3399-android10.0-bundle.git .bundle


3. If the download warehouse fails, the current bundle warehouse is about 1.4G, so there may be stuck or failed problems during synchronization. You can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3399-android10.0-bundle.7z  -r -o. && mv rk3399-android10.0-bundle/ .bundle/


4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update


5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```
