* 单独编译 kernel：

```
cd ~/proj/Station-P1-Pro/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/Station-P1-Pro/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/Station-P1-Pro/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/Station-P1-Pro/
./FFTools/make.sh -j8
```