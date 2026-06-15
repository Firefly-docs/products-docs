### Download Android SDK

Due to the larger SDK, you can choose the cloud disk to download Firefly-RK3399_Android8.1_xxx.7za from the download page (you can directly use the Firefly-RK3399 source package):

[Android8.1 SDK Source]

After the download is complete, verify the MD5 code before decompression:
```
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.001
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.002
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.003

48b4cfcb538cdc39ea53349989691617  firefly_rk3399_android8.1_git_20211217.7z.001
05776d4966d92efdcb88170355d8fbd4  firefly_rk3399_android8.1_git_20211217.7z.002
c83f54d89b4cde6eb620b92cd2e99a05  firefly_rk3399_android8.1_git_20211217.7z.003
```

Then unzip:
```
mkdir -p ~/proj/roc-rk3399-pc
cd ~/proj/roc-rk3399-pc
7z x ~/firefly_rk3399_android8.1_git_20211217.7z.001 -r -o.
git reset --hard
```

Update the remote warehouse:
```
git remote rm origin
git remote add gitlab git@gitlab.com:TeeFirefly/firenow-oreo-rk3399.git
```

Synchronize the source code from gitlab:
```
git pull gitlab firefly-rk3399:firefly-rk3399
```

You can also view the source code online at the following address:
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)

[Android8.1 SDK Source]: http://en.t-firefly.com/doc/download/52.html#other_100