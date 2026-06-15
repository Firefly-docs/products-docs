* 单独编译 kernel：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/AIO-3399C/
./FFTools/make.sh -j8
```