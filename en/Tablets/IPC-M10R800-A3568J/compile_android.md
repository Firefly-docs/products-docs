* Compile the kernel separately：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -k -j8
```

* Compile the uboot separately：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -u -j8
```

* Compile Android upper layer separately：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -a -j8
```

* Compile ubooot, kernel, Android at the same time：

```
cd ~/proj/IPC-M10R800-A3568J/
./FFTools/make.sh -j8
```