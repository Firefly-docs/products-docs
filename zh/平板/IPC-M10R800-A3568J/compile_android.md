* 单独编译 kernel：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -j8
```