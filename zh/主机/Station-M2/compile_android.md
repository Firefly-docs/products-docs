* 单独编译 kernel：

```
cd ~/proj/Station-M2/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/Station-M2/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/Station-M2/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/Station-M2/
./FFTools/make.sh -j8
```