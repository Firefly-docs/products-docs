### Download Firefly_Linux_SDK sub-volume compressed package

**Since the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of sub-volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source code package in the following ways:**[Firefly_Linux_SDK Source]

After downloading, follow the README document:

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

### Update Firefly_Linux_SDK

You can use the following command to update the SDK later
```
.repo/repo/repo sync -c --no-tags
```

[Firefly_Linux_SDK Source]: http://en.t-firefly.com/doc/download/52.html#other_144