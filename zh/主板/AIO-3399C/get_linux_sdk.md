### 下载 Firefly_Linux_SDK 分卷压缩包

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包SDK。用户可以通过如下方式获取 Firefly_Linux_SDK源码包：**[Firefly_Linux_SDK源码包]

下载完成后按照 README 文档操作：

```
└── rk3399_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3399_linux_release_xxx.sdk.split00
    │   ├── rk3399_linux_release_xxx.sdk.split01
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh

```

### 更新 Firefly_Linux_SDK

后续可以使用以下命令更新 SDK
```
.repo/repo/repo sync -c --no-tags
```

[下载链接]: http://www.t-firefly.com/doc/download/page/id/54.html
[Firefly_Linux_SDK源码包]: http://www.t-firefly.com/doc/download/page/id/54.html#other_186