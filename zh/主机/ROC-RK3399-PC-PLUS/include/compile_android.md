* 单独编译 kernel：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/ROC-RK3399-PC-PLUS/
./FFTools/make.sh -j8
```